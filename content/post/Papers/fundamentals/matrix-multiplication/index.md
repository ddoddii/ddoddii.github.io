---
title: "[Review] - Anatomy of High-Performance Matrix Multiplication"
date: 2025-02-25
draft: false
summary : "BLAS(gemm, gemv) 연산 최적화 방법들"
tags: ["BLAS","gemm"]
categories : ["Papers"]
slug : "matrix-multiplication-review"
toc : true
katex : true
markup: 'mmark'
---

{{< katex >}}

## BLAS란?

BLAS는 **Basic Linear Algebra Subprograms** 의 약자로  벡터 덧셈, 스칼라 곱셈, 내적 연산, 선형 조합, 행렬 곱셈과 같은 일반적인 선형 대수 연산을 수행하는 저수준 루틴을 정의하는 명세이다.  BLAS는 선형 대수 라이브러리에서 사실상의 표준 저수준 루틴으로 사용되며, C 언어("CBLAS 인터페이스")와 Fortran("BLAS 인터페이스") 모두에 대한 바인딩을 제공한다.

BLAS 명세는 일반적이지만, 실제 BLAS 구현은 특정 하드웨어에서 성능을 극대화하도록 최적화되는 경우가 많다. 따라서 BLAS를 활용하면 연산 성능을 크게 향상시킬 수 있다. 또한, BLAS 구현은 벡터 레지스터나 SIMD 명령어와 같은 특수 부동소수점 연산 하드웨어를 적극적으로 활용하여 연산 속도를 높인다.

대부분의 선형 대수 루틴을 제공하는 라이브러리는 BLAS 인터페이스를 준수하며, 이를 통해 라이브러리 사용자는 특정 BLAS 라이브러리에 종속되지 않는 프로그램을 개발할 수 있다. 다양한 하드웨어 플랫폼을 대상으로 여러 BLAS 라이브러리가 개발되었다. 예를 들어, **cuBLAS**(NVIDIA GPU, GPGPU), **rocBLAS**(AMD GPU), **OpenBLAS** 등이 있다. CPU 기반 BLAS 라이브러리로는 OpenBLAS, BLIS(BLAS-like Library Instantiation Software), Arm Performance Libraries, ATLAS, Intel Math Kernel Library(iMKL) 등이 있다. 

### BLAS 등장 배경

수치 해석 프로그래밍이 발전하면서, 고급 서브루틴 라이브러리가 유용하게 활용되기 시작하였다. 이러한 라이브러리는 근 찾기(root finding), 행렬 역행렬 계산, 연립 방정식 풀이와 같은 일반적인 고수준 수학 연산을 수행하는 서브루틴을 포함하고 있었다. 당시 가장 널리 사용된 프로그래밍 언어는 FORTRAN이었다.

가장 대표적인 수치 해석 라이브러리는 **Scientific Subroutine Package(SSP)** 였다. 이러한 서브루틴 라이브러리는 프로그래머가 특정 문제 해결에 집중할 수 있도록 도와주며, 이미 잘 알려진 알고리즘을 매번 새롭게 구현하는 번거로움을 줄여주었다.

또한, 라이브러리에서 제공하는 루틴은 평균적인 구현보다 더 정교하게 최적화되어 있었다. 예를 들어, 행렬 알고리즘은 완전 피벗팅(full pivoting) 기법을 사용하여 보다 높은 수치 정확도를 제공할 수 있었다. 뿐만 아니라, 라이브러리는 연산을 더욱 효율적으로 수행하도록 설계되었다. 예를 들어, 라이브러리에는 상삼각 행렬(upper triangular matrix)을 푸는 특화된 루틴이 포함될 수 있다.

이러한 라이브러리는 일부 알고리즘에 대해 단정밀도(single-precision) 및 배정밀도(double-precision) 버전을 제공하여 다양한 정밀도 요구 사항을 충족할 수 있도록 하였다.

초기에는 이러한 서브루틴들이 저수준 연산을 수행하기 위해 하드코딩된 루프(hard-coded loops) 를 사용하였다. 예를 들어, 행렬 곱셈을 수행하는 서브루틴은 일반적으로 세 개의 중첩 루프(nested loops)를 포함하였다.

선형 대수 프로그램에는 여러 공통적인 저수준 연산이 존재하며, 이를 "커널(kernel) 연산" 이라고 부른다. 여기서의 "커널"은 운영체제의 커널과는 무관하다. 1973년부터 1977년까지 여러 핵심적인 커널 연산이 식별되었으며, 이후 수학 라이브러리가 호출할 수 있는 서브루틴으로 정의되었다.

이러한 커널 호출 방식은 기존의 하드코딩된 루프보다 여러 가지 장점을 제공하였다.

- 가독성 향상: 라이브러리 코드가 더 읽기 쉬워짐
- 버그 발생 확률 감소: 반복적인 코드 작성이 줄어듦
- 성능 최적화 가능: 커널 구현을 특정 하드웨어에 맞춰 최적화 가능

1979년, 스칼라 및 벡터 연산을 수행하는 "**레벨-1 BLAS(Basic Linear Algebra Subroutines)**" 명세가 발표되었다. BLAS는 선형 대수 연산을 효율적으로 수행하기 위한 표준으로 자리 잡았으며, 이를 기반으로 한 LINPACK 라이브러리가 개발되었다.

### BLAS의 확장과 성능 최적화

BLAS 추상화 계층은 높은 성능을 위한 커스터마이징을 가능하게 하였다. 예를 들어, LINPACK은 범용 라이브러리로, 여러 머신에서 수정 없이 실행할 수 있었다. 그러나 특정 하드웨어에서 성능을 극대화하려면 해당 머신에 최적화된 BLAS 버전을 사용해야 했다.

컴퓨터 아키텍처가 발전하면서 벡터 프로세서(vector processor) 가 등장하였으며, 벡터 프로세서를 위한 BLAS는 빠른 벡터 연산을 활용할 수 있도록 설계되었다. (비록 벡터 프로세서는 이후 점차 사라졌지만, 현대 CPU에서는 벡터 명령어가 BLAS 루틴의 성능 최적화를 위해 필수적인 요소로 자리 잡았다.)

하드웨어의 발전에 따라 BLAS도 확장되었다.

- 1984~1986년: 레벨-2 BLAS가 추가되었으며, 이는 벡터-행렬 연산(vector-matrix operations) 을 포함하였다.
- 1987~1988년: 레벨-3 BLAS가 도입되었으며, 이는 행렬-행렬 연산(matrix-matrix operations) 을 수행하도록 설계되었다.
  - 레벨-3 BLAS는 블록 분할(block-partitioned) 알고리즘을 장려하였다.
  - LAPACK 라이브러리는 레벨-3 BLAS를 기반으로 구현되었다.

초기의 BLAS는 조밀 행렬(dense matrix) 및 벡터 연산만을 다루었다. 이후 **희소 행렬(sparse matrix)** 에 대한 확장도 이루어졌으며, 보다 다양한 응용 분야에서 활용될 수 있도록 발전해 왔다.

### BLAS Functionality

<u><span style="font-size:130%">**Level 1 BLAS : 벡터 연산 (O(n))**</span></u>

Level 1 BLAS는 벡터 연산(vector operations)만을 수행하며, 실행 시간 복잡도가 **O(n)** 이다. 즉, 입력 크기 \\(n\\) 에 대해 연산 시간이 선형적으로 증가한다.

<span style="font-size:110%">대표적인 연산</span>

1. **scalar multiplication**

$$y \leftarrow \alpha x $$

- 벡터 \\(x\\) 의 각 원소에 스칼라 \\(\alpha \\) 를 곱하여 벡터 \\(y\\) 를 생성하는 연산이다.


2. **vector addition**

$$y \leftarrow x + y $$

- 두 벡터의 원소별 덧셈을 수행한다.

3. **Generalized Vector Addition(AXPY)**

$$y \leftarrow \alpha x + y $$

- AXPY("A times X Plus Y")라고 불리는 연산이다.
- 벡터 \\(x\\) 의 각 원소에 스칼라 \\(\alpha \\) 를 곱한 후, 벡터 \\(y\\) 에 더한다.
- 실행시간은 벡터 크기 \\(n\\) 에 비례하여 \\(O(n)\\)이다.


4. **dot product**

$$s = x^ \intercal y$$

- 두 벡터의 원소별 곱을 계산한 후, 그 합을 구한다.
- 벡터 길이에 비례하여 \\(O(n)\\) 의 시간 복잡도를 가진다.

5. **vector norm**

$$\|x\| = \sqrt{x_1^2 + x_2^2 + \cdots + x_n^2}$$

- 벡터의 크기를 측정하는 연산이다.

<u><span style="font-size:130%">**Level 2 BLAS : 행렬-벡터 연산 (O(n²))**</span></u>

Level 2 BLAS는 행렬-벡터 연산(matrix-vector operations) 을 포함하며, 실행 시간 복잡도가 **O(n²)**이다. 즉, 입력 크기 n 에 대해 연산 시간이 제곱 비례(quadratic time complexity) 로 증가한다.

<span style="font-size:110%">대표적인 연산</span>

1. **General Matrix-Vector Multiplication, GEMV**

$$y \leftarrow \alpha Ax + \beta y $$

- 행렬 \\(A\\) 와 벡터 \\(x\\)의 곱을 계산하고, 벡터 \\(y\\)에 스칼라 \\(\beta \\)를 곱한 값이다. 

2. **Triangular Solve, TRSV(선형방정식 풀이)**

$$Tx = y$$

- \\(T\\)가 삼각행렬(triangular matrix)일때, 연립방정식 \\(Tx=y\\)를 풀어 \\(x\\)를 계산한다.
- 이 연산은 O(n²)의 복잡도를 가진다.

<u><span style="font-size:130%">**Level 3 BLAS : 행렬-행렬 연산 (O(n³))**</span></u>

Level 3 BLAS는 행렬-행렬 연산(matrix-matrix operations) 을 수행하며, 실행 시간 복잡도가 **O(n³)** 이다. 즉, 입력 크기 n 에 대해 연산 시간이 세제곱 비례(cubic time complexity) 로 증가한다.

<span style="font-size:110%">대표적인 연산</span>

1. **General Matrix-Matrix Multiplication, GEMM**

$$C \leftarrow \alpha AB + \beta C $$

- 행렬 \\(A\\) 와 \\(B\\) 를 곱한 후, 스칼라 \\(\beta\\)를 곱한 행렬 \\(C\\)를 더한다.
- 행렬 \\(A\\)가 \\(n × n\\) 크기라면 연산량이 O(n³)



2. **삼각 행렬을 이용한 행렬 변환**

$$B \leftarrow \alpha T^{-1} B$$

- 삼각행렬 \\(T\\)의 역행렬을 이용해 행렬\\(B\\)를 변환하는 연산이다.

Level 3 BLAS에서는 블록 분할(block partitioning) 기법을 활용하여 메모리 계층(cache hierarchy)을 최적화할 수 있다.

이제 BLAS의 세 가지 레벨을 명확하게 이해했을 것이다. Level 1은 단순 벡터 연산, Level 2는 행렬-벡터 연산, Level 3는 행렬-행렬 연산으로, 연산 규모가 커질수록 실행 시간 복잡도가 증가한다. 하지만 현대의 최적화된 BLAS 구현을 활용하면 CPU와 GPU의 연산 성능을 극대화할 수 있다. 현대 BLAS 구현에서는 SIMD(Vectorization) 및 병렬 연산을 활용하여 성능을 극대화한다.

행렬 곱셈(matrix multiplication)은 수많은 과학 및 공학 응용에서 필수적인 연산이며, 특히 Level 3 BLAS의 대부분의 연산이 GEMM(General Matrix-Matrix Multiplication) 루틴을 기반으로 구현된다. 단순한 행렬-벡터 곱셈(matrix-vector multiplication)을 반복하는 방식보다 더 빠른 알고리즘이 존재하며, 이는 BLAS 구현에서 최적화의 핵심 대상이 된다.

## GEMM 최적화 전략

<span style="font-size:130%">**1. 블록 행렬 분할(Blocking)**</span>

(1) **블록 행렬 분할 기법**

행렬 곱셈을 최적화하는 주요 기법 중 하나는 블록 행렬(block matrix) 분할을 활용하는 것이다. 행렬 A,B 를 작은 블록 행렬로 나누고, 재귀적으로 GEMM을 수행하면 성능을 향상시킬 수 있다.

원래 GEMM은 아래와 같다.

$$C \leftarrow \alpha AB + \beta C $$

여기서 A,B를 블록 행렬로 분할하면:

$$C_{i,j} \gets \sum_k A_{i,k} B_{k,j} + C_{i,j}$$

이러한 방식은 **메모리 접근의 지역성(locality of reference)** 을 향상시키며, 캐시 효율성을 극대화할 수 있도록 돕는다.

(2) **\\(\beta \\) 매개변수의 역할**

GEMM에서 제공되는 \\(\beta \\) 파라미터는 이전 블록 행렬 곱 결과를 누적(accumulate)하는 역할을 한다. 즉, 현재 블록 연산 결과를 기존 행렬 \\(C \\) 에 더할 수 있도록 해주며, 이로 인해 캐시 효율성이 향상된다.
특히, 특정 구현에서는 \\(\beta =1 \\) 인 경우를 최적화하여 추가적인 곱셈을 제거할 수 있다. 즉, \\(C \\) 에 대해 추가적인 행렬 연산을 최소화하면서 성능을 향상시키는 것이다.

(3)  **다중 캐시 계층(L1, L2, L3)에 대한 블로킹**

현대 시스템에는 여러 계층의 캐시 메모리(L1, L2, L3)가 존재하며, 각 계층에서 데이터를 효과적으로 활용하기 위해 이중 블로킹(double blocking) 을 적용할 수 있다.
즉, 블록 분할을 한 번 적용한 후, 블록의 순서를 다시 최적화하여 캐시 계층별로 최적의 성능을 발휘할 수 있도록 한다.이러한 최적화 기법은 ATLAS, OpenBLAS, BLIS 등의 라이브러리에서 적극적으로 활용되고 있다.


<span style="font-size:130%">**2. GotoBLAS 및 최신 최적화 기법**</span>

최근에는 GotoBLAS에서 제안된 최적화 기법이 ATLAS보다 더 우수한 성능을 보인다. 주요 개선점은 다음과 같다.

(1) **L2 캐시를 중심으로 블로킹 수행**

ATLAS는 L1, L2 캐시를 모두 고려하여 블로킹을 수행하지만, GotoBLAS는 L2 캐시만을 최적화 대상으로 삼는 것이 더 효율적이라는 점을 발견하였다.
L2 캐시 크기에 맞춰 블록 크기를 설정하면, 더 적은 메모리 이동으로 높은 성능을 유지할 수 있다.

(2) **연속적인 메모리 복사(Copying to Contiguous Memory) 최적화**

메모리를 연속적으로 배치하면 **TLB misses(Translation Lookaside Buffer Misses)** 를 줄일 수 있다.
GotoBLAS는 데이터를 미리 복사하여 연속적인 메모리 블록을 유지함으로써, 행렬 연산의 속도를 극대화하였다.
이러한 최적화 기법은 GotoBLAS, OpenBLAS, BLIS 등의 최신 BLAS 라이브러리에 포함되어 있다.


## Anatomy of High-Performance Matrix Multiplication

이제 비교적 오래된 논문에서, 매트릭스 연산 최적화를 어떻게 제시했는지 살펴보자.


### A Layered Approach to GEMM

\\(m × n\\) 크기인 행렬 \\(X\\) 를 보자. \\(X\\) 를 아래와 같이 행렬로 나눈 경우를 생각하자:

<img width="410" alt="Image" src="https://github.com/user-attachments/assets/49934103-1692-4724-96e1-d8f51eef9f20" />

이때, \\(X_j\\) 는 \\(n_b\\)개의 열을 가지고 있고, \\(\check(X_i)\\)는 \\(m_b\\)개의 행을 가지고 있다. 

매트릭스 연산을 각각 작은 매트릭스(submatrices)로 분할하여 생각해보자. 여기서 저자들은 이러한 형태의 연산이 매우 자주 일어나는 것을 발견했다. 예를 들어서, LAPACK가 지원하는 여러 가지 연산들은 GEPP, GEMP, GEPM 으로 이루어져 있다. 


<img width="635" alt="Image" src="https://github.com/user-attachments/assets/3a1769b3-895b-4a86-a556-648f64435503" />


<img width="1144" alt="Fig.4" src="https://github.com/user-attachments/assets/093959f6-b0a4-4310-ade2-c7f7d00b85df" />


Fig. 4를 보면, 일반적인 GEMM 연산이 어떻게 Fig. 2에서 설명된 다양한 특수한 형태의 연산으로 체계적으로 분해될 수 있는지를 보여준다.

즉, GEMM 연산은 여러 개의 GEPP, GEMP, GEPM 호출로 분해될 수 있으며,이 연산들은 다시 GEBP, GEPB, GEPDOT 같은 커널 함수들로 더 세분화될 수 있다.

이 접근 방식의 핵심 아이디어는 **가장 하위 계층의 세 가지 커널(GEBP, GEPB, GEPDOT)이 높은 성능을 달성하면, 나머지 GEMM 연산들도 자연스럽게 높은 성능을 낼 수 있다** 는 것이다.

<img width="501" alt="Fig.5" src="https://github.com/user-attachments/assets/258d0580-63f6-4de3-a2c7-0fd672378985" />

Fig. 5에서는 일반적인 GEMM 연산을 작은 블록 단위로 나누어 수행하는 방식을 설명한다.

1. **행렬 C,A,B 의 블록 분할 (Partitioning into Submatrices)**

이를 위해 행렬 C,A,B 를 작은 서브 행렬(submatrices)로 나누는데, 다음과 같이 블록 단위를 정의한다:

<img width="582" alt="Image" src="https://github.com/user-attachments/assets/03323ecc-2bfd-4821-9855-c8abf97c300e" />


- \\(C_{ij} \in \mathbb{R}^{m_c × n_r}  \\) → 결과 행렬 \\(C\\) 의 개별 블록
- \\(A_{ip} \in \mathbb{R}^{m_c × k_c}  \\) → 행렬 \\(A\\) 의 블록
- \\(B_{pj} \in \mathbb{R}^{k_c × n_r}  \\) → 행렬 \\(B\\) 의 블록

즉, \\(C\\) 행렬의 각 원소 \\(C_{ij}\\)는 다음과 같이 계산된다: 

$$C_{ij} += A_{ip}B_{pj}$$

여기서, 블록 크기 \\(m_c, n_r, k_c\\) 는 논문 후반부에 더 자세히 설명된다. 

2. **3중 중첩 루프(Triple-Nested Loop)와 GEMM 분해 과정**

Fig. 5에서는 GEMM 연산을 작은 블록으로 나누어 수행하는 과정을 3중 중첩 루프 형태로 나타낸다:

```
for p = 1 : K
    for i = 1 : M
        for j = 1 : N
            C_ij += A_ip * B_pj
        endfor
    endfor
endfor
```

각 루프가 수행하는 역할은 다음과 같다:

(1) j 루프`(for j = 1:N)` -> 행렬 \\(B\\)를 \\(n_r\\)크기의 블록으로 분할하여 부분연산 수행

(2) i 루프`(for i = 1:M)` -> 행렬 \\(A\\)를 \\(m_c\\)크기의 블록으로 분할하여 부분연산 수행

(3) p 루프`(for p = 1:K)` -> 행렬 \\(A\\),\\(B\\)를 \\(k_c\\)크기의 블록으로 나누어 순차적으로 처리


즉, GEMM 연산을 바로 수행하는 것이 아니라, 블록 단위로 나누어 작은 크기의 행렬 곱셈을 여러 번 수행함으로써 연산을 최적화한다.

3. **GEMM 최적화 계층 (Layered Optimization)**

Fig. 5에서는 GEMM 연산을 더욱 세분화하여 최적화하는 계층적인 접근 방식을 설명한다.
이 과정은 다음과 같이 단계적으로 나뉜다:

- GEBP (Generalized Edge Block Panel)
  - 가장 작은 블록 단위 연산 수행
  - 작은 패널(panel) 크기의 서브 행렬을 사용하여 행렬 연산을 수행
  - 캐시 친화적인(blocking) 연산을 수행하여 성능 최적화

- GEPP_VAR1 (Generalized Edge Panel Panel - Variant 1)
  - 중간 크기의 블록을 사용하여 연산을 수행
  - 여러 개의 블록을 조합하여 보다 큰 연산을 처리

- GEMM_VAR1 (Generalized Matrix-Matrix Multiplication - Variant 1)
  - 전체 행렬을 작은 블록 단위 연산으로 완성
  - 최종적으로 GEMM 연산을 블록 기반으로 수행

### High-Performance GEBP, GEPB, GEPDOT

이제 GEBP, GEPB, GEPDOT 연산을 고성능으로 구현하는 방법을 살펴보자.우선, 메모리 계층 간 데이터 이동 비용(cost of moving data) 을 분석하는 것으로 시작하며, 이후 보다 실제적인 모델을 추가하여 GEPP, GEMP, GEPM 알고리즘으로 확장할 것이다.

#### 1. Basics 

Fig. 6 (왼쪽)에서는 간단한 메모리 계층 모델을 보여준다. RAM과 레지스터 사이에 하나의 캐시(Cache) 계층을 추가한 모델이다. 이 모델을 활용하면, GEBP, GEPB, GEPDOT의 가장 간단한 성능 최적화를 설명할 수 있다.

<img width="626" alt="Fig.6" src="https://github.com/user-attachments/assets/9f835d18-1a50-4494-aad5-0085cb551b47" />

우선 \\(A \in \mathbb{R}^{m_c × k_c}  \\), \\(B \in \mathbb{R}^{k_c × n}\\), \\(C \in \mathbb{R}^{m_c × n}\\) 인 GEBP 를 보자. 아래와 같이 행렬 B,C를 분할하자:

$$B = (B_0 | B_1 | \cdots | B_{N-1}) and C = (C_0 | C_1 | \cdots | C_{N_1})$$ 

그리고 3가지 가정을 하는데 :

\\(Assumption (a)\\). \\(m_c,k_c\\) 는 충분히 작아서, \\(A\\) 및 행렬 \\(B,C\\)  의 \\(n_r\\) 번째 열인 \\(B_j,C_j\\) 가 둘다 캐시에 완전히 적재가능해야 한다.

\\(Assumption (b)\\). 만약 \\(A,C_j, \ and \ B_j\\)가 캐시에 있으면 \\(C_j := AB_j + C_j\\) 는 CPU의 최대 속도로 연산될 수 있다.

\\(Assumption (c)\\). 만약 \\(A\\)가 캐시에 있으면 더 이상 필요 없어질 때까지 캐시에 있는다. 

이러한 가정들 하에, Fig.7 의 접근에서 GEBP 알고리즘이 메인 메모리에서 캐시로 데이터 이동 비용을 최소화할 수 있다.

<img width="454" alt="Fig.7" src="https://github.com/user-attachments/assets/460f7eb3-dd9b-4ea6-ac15-512fa9e9a3bd" />

<img width="854" alt="GEBP help" src="https://github.com/user-attachments/assets/d5e55066-ea6a-4c4f-9eb6-cafeddd64dda" />



## Reference

- https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms#Level_3
- Goto, Kazushige, and Robert A. Van De Geijn. “Anatomy of High-Performance Matrix Multiplication.” ACM Transactions on Mathematical Software 34, no. 3 (May 2008): 1–25. https://doi.org/10.1145/1356052.1356053.
