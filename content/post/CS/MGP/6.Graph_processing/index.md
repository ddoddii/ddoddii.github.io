---
title: "그래프 구조를 더 효율적으로 저장하는 방법들"
date: 2024-04-21
draft: false
summary : "Compressed Sparse Row, Compressed Sparse Column 에 대한 소개"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 6
slug : "graph-processing"
toc : true
katex : true
markup: 'mmark'
---

본 포스팅에서는 큰 용량을 차지하는 그래프 구조를 좀 더 효율적으로 저장하는 방법들에 대해 알아보겠습니다. 

## Sparse Matrix-Vector Multiplication(SpMV)

대개 큰 매트릭스에서, 실제 0이 아닌 값을 가진 원소는 매우 적습니다. (<1%) 그러면 매트릭스 전체를 다 저장하는 것은 비효율적입니다. 하지만 0이 아닌 값들만 저장하려면 여러 가지 문제가 있습니다. 첫번째로, 실제 값들의 위치를 알아야 합니다. 두번째로, dense matrix 과는 다르게 메모리에 대해 비규칙적으로 접근해야 합니다. 세번째로, dense matrix 일 때 병렬화보다 sparse matrix 일 때 병렬화가 어렵습니다. 왜냐하면 0이 아닌 값들이 균등하게 분포되어 있지 않아서, workload balancing 이 어렵기 때문입니다. 

## Graphs

그래프 구조는 정말 다양한 영역에서 사용됩니다. 예를 들어 트위터의 팔로워, 서로 연결되어 있는 웹페이지, 논문 인용 등이 있습니다. 그렇지만 그래프는 매우 크고 구조화되어 있지 않는 데이터입니다. 따라서 matrix 형태로 표현하곤 합니다. 

그래프로 할 수 있는 것들은, 아래의 경우들이 있습니다. 
- 최단 경로
- 페이지 랭크
- sparse matrix 계산

그래프 구조를 나타내기 위해 우선 **인접 행렬(adjacency matrix)** 을 만들 수 있습니다. 


<img width="331" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/58e36747-6b8b-4170-aa45-50c083b39393">


<img width="299" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8790d8c5-3920-4491-87b3-f2756a583a68">

- **Pagerank PULL** 계산하기 
		PULL은 내 노드를 가리키는 노드들이 몇개인지 (팔로워) 계산합니다. 

- **Pagerank PUSH** 계산하기
		PUSH는 내 노드가 가리키는 노드들이 몇개인지(팔로잉) 계산합니다. 

인접행렬을 사용해서 **Pagerank PULL** 을 계산해보면, 기준 노드가 dest 노드이므로, row 를 X 에 행렬곱 하는 것과 같습니다. 2번 노드의 PULL 을 계산할 때는 아래와 같이 계산할 수 있습니다.

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cfbe9c63-45a5-4b47-9fab-db83812ff2bc">

**Pagerank PUSH** 는 기준 노드가 src 노드이므로, col 을 기준으로 행렬곱 하는 것과 같습니다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/546ba820-90ae-482e-9abe-8360483a3fbd">

이전 포스팅에서 다루었던 매트릭스 곱과 같이 다루면 안될까요? 하지만 그래프가 점차 커지면, 오직 1% 미만의 원소들만 0이 아닌 값들입니다. 따라서 다른 방법으로 다루어야 합니다. 

## CSR / CSC

CSR 는 **Compressed Sparse Row**, CSC 는 **Compressed Sparse Column** 입니다. CSR 과 CSC 는 좀 더 효율적으로 거대한 sparse 매트릭스를 저장할 수 있는 방법을 제공합니다. 

### Compressed Sparse Row

위의 인접행렬을 CSR 로 나타내 봅시다. 이때 `row_ptr`, `col_index`, `vertex_data` 3개의 배열이 필요합니다. 다만 여기서는 value 가 1이므로 `vertex_data` 에는 모두 1만 저장됩니다. `col_index` 에는 각 행 내에서 1이 있는 인덱스를 저장합니다. `row_ptr` 는 `col_index` 에서 각 행이 나타내는 지점의 시작점을 나타냅니다. 

예시를 보면 쉽게 이해가 갑니다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3491acd8-1bd4-4c82-91d1-d4cc1b2896ff">

`row_ptr` 에서 2-0 은 row0 에 있는 nonzero 값들의 개수(2) 를 나타내고, 그 다음 2-2 는 row1 에 있는 nonzero 값들의 개수(0) ... 를 나타내는 방식입니다. 실제 nonzero 값의 위치는 `col_index` 를 보면 알 수 있습니다. 

CSC 로 CSR 과 원리는 같고, 열을 기준으로 나타내는 방식입니다. 

CSR 로 **SPMV(Sparse Matrix Vector Multiplication)** 을 계산하면 아래와 같이 코드를 짤 수 있습니다.

```cpp
for (int row=0; row < num_rows; row++) {
	float dot = 0;
	int row_start = row_ptr[row];
	int row_end = row_ptr[row+1];
	for (int elem = row_start; elem < row_end; elem++) {
		dot += data[elem] * x[col_index[elem]];
	}
	y[row] = dot;
}
```

### CSR SPMV Design

CSR 에서 SPMV 를 할때, 멀티쓰레드로 작업을 분배하려면 어떻게 해야 할까요? 우선 가장 간단한 방법은 row 기준으로 쓰레드를 할당하는 것입니다. 그러나 dense matrix 과는 다르게 row 별로 연산량이 다릅니다. 따라서 쓰레드별로 workload 가 불균형해지는 문제가 생깁니다. 또한 메모리 접근 패턴이 그래프 구조마다 달라지기 때문에 일반적으로 최적화하는 기법을 만들 수 없습니다.

### CSR/CSC가 최선일까?

CSR/CSC 는 그냥 매트릭스 형태로 나타냈을 때보다 공간 저장 효율성이 뛰어납니다. 하지만, 업데이트가 일어날 때마다 `row_ptr` 과 `col_index` 를 모두 수정해야 합니다. 
- Space : O(|V|+|E|) (|V|: 노드의 개수, |E| : 엣지의 개수)
- 엣지가 존재하는 지 확인 : O(deg(v))
- 엣지 추가하기 : O(|V|+|E|)
- 엣지 삭제하기 : O(|V|+|E|)
- 노드의 이웃들 찾기 : O(deg(v))
- deg(v) (노드v의 엣지 개수) 구하기 : O(1)

반면 매트릭스는 저장 공간은 엄청나게 많이 차지하지만, 엣지를 추가하고 삭제하는 연산은 O(1) 에 할 수 있습니다. 
- Space : O(|V|^2)
- 엣지가 존재하는 지 확인 : O(1)
- 엣지 추가하기 : O(1)
- 엣지 삭제하기 : O(1)
- 노드의 이웃들 찾기: O(|V|)
- deg(v) 구하기 : O(|V|)

### Edge list (with array)
CSR 말고 다른 방법은 **배열**을 가지고 구현하는 **edge list** 도 있습니다. 
- Space : O(|E|) (|E| : 엣지의 개수)
- 엣지가 존재하는 지 확인 : O(log|E|) (이진탐색 이용)
- 엣지 추가하기 : O(|E|)
- 엣지 삭제하기 : O(|E|)
- 노드의 이웃들 찾기 : O(log|E| + deg(v))
- deg(v) 구하기 : O(log|E| + deg(v))

### Edge list (with linked list)
linked list 로 edge list 를 구현할 수 도 있습니다.  이 경우에는 하나의 노드에 대한 여러 개의 연산을 수행하는 경우에 좋습니다. 알맞는 노드만 찾았다면 노드를 추가/삭제하는 것은 O(1) 에 할 수 있습니다. 
- Space : O(|E|) (|E| : 엣지의 개수)
- 엣지가 존재하는 지 확인 : O(|E|) 
- 엣지 추가하기 : O(|E|)
- 엣지 삭제하기 : O(|E|)
- 노드의 이웃들 찾기 : O(log|E|)
- deg(v) 구하기 : O(log|E|)

### Adjacency list

<img width="402" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a19f2dba-be2e-475c-b821-0b412ab4c227">

여러 개의 linked list 로 구현하는 방법입니다. 포인터가 있는 배열을 사용합니다. 각 노드는 정렬되지 않은 엣지의 리스트들을 가지고 있습니다. 
- Space : O(|V|+|E|) 
- 엣지가 존재하는 지 확인 : O(deg(v)) 
- 엣지 추가하기 : O(1)
- 엣지 삭제하기 : O(deg(v))
- 노드의 이웃들 찾기 : O(deg(v))
- deg(v) 구하기 : O(deg(v))

## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- [sparse matrix (희소 행렬)와 CSR(Compressed Sparse Row)](https://gaussian37.github.io/math-la-sparse_matrix/)
- https://en.wikipedia.org/wiki/Sparse_matrix#:~:text=The%20compressed%20sparse%20row%20
