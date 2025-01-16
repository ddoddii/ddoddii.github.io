---
title: "멀티쓰레드에서 행렬 연산(matmul) 성능 증가시키는 방법들"
date: 2024-04-13
draft: false
summary : "캐시구조에 따른 행렬곱 연산 성능 높이기"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 4
slug : "multithread-matmul"
toc : true
katex : true
markup: 'mmark'
---


## Matrix multiplication basics

분석을 더 쉽게 하기 위해 정방행렬만 고려합시다.

<img width="600" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/3099b478-bdeb-49f5-ab4f-be11cdfcec0f">

\\(C_{ij} =\Sigma_{k} \ A_{ik}B_{kj}\\)

```
for i = 1 to n
	for j = 1 to n
		for k = 1 to n
			C(i,j) = C(i,j) + A(i,k) * B(k,j)
```

이 계산을 어떻게 **병렬화**할까요?

### Try1 : Columnwise block striping

C의 칼럼에 각 쓰레드를 할당하는 방식입니다. 

```
for i = 1 to n
	for j = p to n step P
		for k = 1 to n
			C(i,j) = C(i,j) + A(i,k) * B(k,j)
```

<img width="385" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/96742edb-015d-4ff9-b017-418bc7f8202b">

C의 한개의 칼럼을 계산하기 위해서는, A 전체를 읽고 B의 칼럼 한개를 읽어야 합니다. 이때 복잡도는\\(O(N)=N^3\\)입니다. 이 방식으로는, 중복되는 계산이 없으므로 겹치는 것에 대한 고려를 하지 않아도 됩니다. 따라서 락이 필요 없습니다. 

<img width="175" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/fe7effe2-e33a-4580-bbff-0c2434d6af22">

이상적인 CPU 라면 완벽하게 계산되겠지만, 컴퓨터의 하드웨어에는 특별한 저장소인 **캐시**가 있습니다. 

<img width="309" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/67a79d60-11ca-4f92-ae35-6e15e1d9a438">

즉 C의 각 칼럼을 서로 다른 쓰레드들이 연산을 한다고 해도, C의 행에 캐시 라인이 있습니다. 만약 두개의 쓰레드가 동시에 하나의 행(캐시 라인) 에 쓰기 연산을 하면, 캐시 동기화 오버헤드가 생기게 됩니다. 만약 캐시 업데이트가 제대로 안 이루어지면 **false sharing** 문제가 생길 수 있습니다. 이러한 캐시 동기화는 상당한 성능 저하로 이루어질 수 있습니다. 


### Try2 : Rowwise block striping

<img width="178" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/0e794f1b-0295-4b30-83b9-601f03faf22e">

그렇다면 C의 각 행에 쓰레드를 할당하면 어떨까요? 이때도 중복되는 연산이 없고, 열에 쓰레드를 할당했을 때 생기는 false sharing 문제도 해결이 됩니다.

여기서는 A의 행 1줄을 읽을 때마다, B의 모든 칼럼을 읽어야 합니다. 이때 칼럼을 읽는 것은 메모리 구조에 따라 계속해서 캐시 미스가 발생하므로, 행 방향으로 데이터를 읽는 것이 캐시 히트가 될 확률이 높습니다. 따라서 B을 Transpose(전치) 시키면,\\(B^T\\)는 행 방향으로 데이터를 읽어올 수 있습니다. 하지만, 여기서도 행렬을 전치시키는데는 많은 오버헤드가 듭니다. 

다른 옵션은 없을까요? 우선 **싱글 쓰레드 연산에서 최적화**부터 해봅시다. 

## Cache-aware performance analysis

**캐시의 계층 구조**에서, L1 캐시는 대략 32KB ~ 64KB 의 사이즈입니다. 여기에는 대략 8K ~ 16K 의 floating point 숫자(1개당 4B)를 담을 수 있습니다. 이때 대략 128 * 128 크기의 매트릭스까지 저장할 수 있습니다. (128 * 128 * 4B = 64KB) 하지만 L1 캐시에는 다른 것들도 저장해야 하므로, 대부분의 경우 N = 64가 최대가 됩니다. 

<img width="600" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/4fa57e01-00c0-4d42-8ab9-4d40c81e86ff">

### Single-thread performance

행렬 연산을 다시 봅시다. 


```
for i = 1 to n
	for j = 1 to n
		for k = 1 to n
			C(i,j) = C(i,j) + A(i,k) * B(k,j)
```


<img width="277" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/e399b26f-8163-43b1-9f7b-883e5ab3d055">

i : 1, j : 1 이고 k : 1->N 인 경우에, 처음 A,B 에 있는 데이터를 처음 로드하는 과정에서는 캐시 미스(cold miss) 가 발생합니다. 


<img width="268" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/2b25d1a7-9bff-4eb8-bc28-7d0387821381">

i=1, j=2, k = 1->N 인 경우에는, A는 all hit 이고, B는 또 다시 all miss 입니다. 왜냐하면 A의 첫 열은 아까 처음 과정에서 로드하면서 데이터가 이미 캐시에 불러와졌기 때문입니다. 하지만 B는 열 기반으로 연산하기 때문에 계속해서 메모리에서 데이터를 가져와야 합니다. 

i : 1 , j : 1->N, k : N * (1->N) 인 경우에, A 는 N/c 만큼의 read 가 발생하고 B는\\(N^2\\)/ c 만큼의 read 가 발생합니다. 여기서 c 는 캐시 라인의 사이즈(64B)입니다. 

i 가 2가 되어서 다음 행의 연산이 일어나는 경우에는 B는 캐시를 사용할 수 있을까요? 이때도 capacity miss 때문에 사용하지 못합니다. Capacity miss 는 캐시의 사이즈가 작아서 이전의 데이터가 날라가서 생기는 미스 입니다. A의 경우에도 이제 새로운 캐시 라인이므로, cold miss 가 발생합니다. 

<img width="265" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/ee5f4411-3a06-4b53-a34b-c6550b8d7ec1">

따라서 C의 전체를 계산하면(i : 1->N, j : N * (1 ->N), k :\\(N^2\\)* (1->N)), 
- A :\\(N^2\\)/ c  reads
- B:\\(N^3\\)/ c  reads 
- C는\\(N^2\\)/ c  reads +\\(N^2\\)/ c  writes 
가 발생합니다. 

## Orthogonal cache access

여기서 B에서 정말 j : 1->N 일때 B에서\\(N^2\\)/ c  만큼의 read 를 하는게 맞을까요? 캐시는 하나의 아이템만 읽어오는 것이 아니라 캐시 라인 단위로 읽어 옵니다. 따라서 하나의 요소를 읽을 때 N 말고 16N 만큼을 저장해야 합니다. B를 columnwise 로 읽을 때, 캐시 사이즈가 16N 보다 작다면 전체 B를 읽는 것은\\(N^3\\)/ c 만큼의 read 가 아니라\\(N^3\\) 만큼의 read 를 하게 됩니다. 

<img width="413" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/25f11a90-acbf-44aa-be93-903a4c94d75a">

따라서 조금 더 엄밀하게 조건을 업데이트 하면, 
- A :\\(N^2\\)/ c  reads
- B:\\(N^3\\)/ c  reads  **(if 16N < cache size)**
- C는\\(N^2\\)/ c  reads +\\(N^2\\)/ c  writes 
입니다. 

## Set collision

### Cache set associativity

컴퓨터 아키텍쳐 시간에 캐시의 **set associativity** 에 대해 배웠습니다. Set associative 캐시에서는 한 블록이 들어갈 수 있는 자리의 개수가 고정되어 있습니다. set 가 여러 개 있을 때, 캐시가 들어가는 자리는 DRAM 의 블럭 위치에 따라 들어갑니다. 8 way set associative 캐시는 각 set 가 8개의 캐시 라인을 저장할 수 있음을 의미합니다. 

데이터를 메모리에서 읽어올 때, 캐시 컨트롤러는 데이터가 저장될 set 를 결정합니다. 이것을 index bits 라고 합니다. 주소의 남은 부분인 tag bits 는 set 내에서 데이터를 식별하기 위해 사용합니다.

만약 메모리에서 읽어오는 데이터가 모두 같은 set 에 할당되면, 캐시를 유용하게 사용할 수 없으므로 **cache set thrashin**g 의 문제가 생깁니다. 


<img width="287" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/501de061-c8ca-477b-966b-83e2063f73e7">

set associativity 와 thrashing 문제는 [여기](https://github.com/ddoddii/OS-CA-Study/tree/main/Computer%20Architecture/3.%20Memory%20Hierarchy#software-optimization-via-blocking)에서 더 자세히 공부했습니다. 



다시 매트릭스 문제로 돌아와서 이게 어떠한 문제를 발생시키는지 봅시다. 
- N 이\\(2^n\\)의 배수일 경우 (e.g N = 128), 128 * 4B = 512B = 8 캐시 라인들을 가지고 있습니다. 
- Haswell CPU 는 32KB L1D 의 8-way set associativity 를 가지고 있습니다.
- 캐시 라인 사이즈는 64bytes 입니다.
- 총 캐시 set 의 수는 32KB / (8 ways * 64bytes) = 64 sets 입니다. 

만약 데이터가 64개의 sets 중 오직 8개의 set 로만 매핑될 경우, 캐시가 32KB의 공간이 있음에도 불구하고 8개의 set 만 활용되어, 캐시 수용 공간이 8 sets * 8 ways * 64 bytes = 4KB 가 되고, 이것은 실제 캐시 사이즈의 1/8 밖에 되지 않습니다. 

 따라서 앞에서 봤던 16N < cache size 의 조건이 아니라 **16N < cache size / 8** 로 업데이트 해야 합니다. (숫자 8이 중요한 것이 아닙니다. 이 숫자는 하드웨어 특징에 따라 달라질 수 있습니다.) 중요한 것은 하드웨어에 있는 addressing scheme 때문에 전체 캐시가 다 활용되지 않을 수 도 있다는 점입니다!

만약 **fully associative cache** 인 경우는 어떨까요? 여전히 캐시 전체를 다 활용하지 않을까요? fully associative cache 는 캐시가 어느 위치든 갈 수 있는 구조입니다. 따라서 이 구조에는 캐시 전체를 다 활용할 수 있습니다. 하지만, 대부분의 CPU는 fully associative cache를 채택하고 있지 않습니다. 왜냐면 하드웨어가 flexible addressing 을 하는 비용이 굉장히 크기 때문입니다. 

### Solution 

N 이\\(2^n\\)의 배수가 아니면 됩니다. N이 \\(2^n\\) 의 배수가 아닐 때 모든 캐시 라인을 활성화할 수 있는 이유는 캐시 라인이나 셋을 균일하게 사용하지 않게 되는 '**cache thrashing**' 현상을 방지하기 때문입니다.

캐시 메모리에서는 주소가 캐시 세트에 매핑될 때 특정 비트를 사용합니다. N이 \\(2^n\\) 의 배수일 경우, 주소의 일부분이 캐시 세트를 선택하는 데 사용되는 인덱스가 되고, 이 인덱스는 주소 패턴에 따라 특정 캐시 세트에 데이터가 집중될 수 있는 가능성이 있습니다. 이것은 캐시 세트들 사이에 데이터 분포가 불균등하게 되고, 결과적으로 캐시의 일부 세트는 자주 사용되지 않게 됩니다.

반면, N이 \\(2^n\\) 의 배수가 아닐 때는, 주소를 매핑하는 방식에 조금 더 변동성이 생깁니다. 즉, 주소 패턴이 캐시 세트에 더 불규칙적으로 매핑되어 각각의 캐시 세트가 더 균등하게 사용될 가능성이 증가합니다. 이로 인해 캐시의 모든 세트가 더 활발히 사용될 수 있으며, 이는 캐시 활용도를 높이고 성능을 향상시킬 수 있습니다.

## Padding & transpose

N을 고정하고, 위의 문제들을 해결하는 방법들은 없을까요? **Padding** 과 **Transpose** 가 있습니다. 

### Solution1: Padding

B보다 더 큰 사이즈의 배열을 만든 후, B를 복사합니다. 이때 새로운 배열을 만들고, B를 복사하는 과정에서 오버헤드가 발생합니다. 또한 더 새로운 배열은 큰 용량을 차지하므로 메모리 공간 낭비가 생길 수 있습니다. 하지만 캐시 활용도를 높일 수 있습니다. 

<img width="509" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/7858b86a-4616-46cf-82c5-5a639177cd32">

### Solution2: Transpose
<img width="542" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/6041b89e-4489-498a-934e-35de092360b0">

패딩보다 나은 두번째 방법은 B 행렬를 전치(transpose) 시키는 것입니다. 전치하는 것에도 추가적인 연산이 들지만, 캐시 히트를 늘릴 수 있으므로 B의\\(N^3\\)이었던 read 를\\(N^3\\)/ c 로 줄일 수 있습니다. 


## Matrix blocking (tiling)

다른 해결책은 일종의 divide-and-conquer 기법을 사용하는 것입니다. L1 캐시 내에 들어갈 수 있도록 매트릭스를 분할하는 것입니다. 작은 블럭으로 나누어 연산을 여러 번 하는 것입니다.

이 경우에는 블럭의 크기, 블럭의 수 등 많은 요인들에 의해 성능이 좌우됩니다. 

<img width="392" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/f7cc0dbf-a3b6-471b-b795-547dce86a3f9">

- A00 *  B00 
	- 각 블럭은 b * b 매트릭스
	- A00 :\\(b^2\\)/ c reads
	- B00 : \\(b^2\\)/ c reads
	- A00과 B00은 캐시에 들어감 
- A01 * B10
	- A01 :\\(b^2\\)/ c reads
	- B10 : \\(b^2\\)/ c reads
	- A01, B10 도 캐시에 들어감 
- A00 * B10
	- A00 :\\(b^2\\)/ c reads
	- B10 : \\(b^2\\)/ c reads
	- 이번에는 A00 전체가 캐시에 들어가지는 못한다고 합시다. 

<img width="264" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/e5a2170d-e576-44fd-b180-66c930ba25cc">

- i : 1 -> N/b
- j : (N/b) * 1 -> N/b
- k : (N/b)^2 * 1 -> N/b

전체 A *  B 에 대해, 
- N/b : 총 블럭의 수
- A :\\((N/b)^{3}\\)*\\(b^2/c\\)=\\(N^3/bc\\)=\\(\frac{N}{b} * N^2/c\\)reads
- B :\\((N/b)^{3}\\)*\\(b^2/c\\)=\\(N^3/bc\\)=\\(\frac{N}{b} * N^2/c\\)reads
- C :\\(N^2/c\\)reads and writes

따라서 b 가 클수록, 더 빠른 연산(matmul) 을 할 수 있는 것을 볼 수 있습니다. 그렇지만 여전히 b 가 캐시에 들어가게끔 설계해야 합니다. 

블러킹으로 인해 성능이 향상되는 것을 [여기](https://github.com/ddoddii/OS-CA-Study/tree/main/Computer%20Architecture/ch.5%20Memory%20Hierarchy#software-optimization-via-blocking)에서 더 자세히 공부했습니다.

## Two-level blocking

Two-level blocking 은 L1 말고 L2 도 추가적으로 사용하는 방식입니다. N이 커질 때 소량의 성능 향상이 있습니다.

<img width="286" alt="image" src="https://github.com/ddoddii/Computer-Science-Study/assets/95014836/690a75b6-d784-4611-8f71-942595dbfb22">




## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- [[컴퓨터 구조] 메모리[4] - Associative Cache](https://ttl-blog.tistory.com/1095)