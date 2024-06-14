---
title: "CUDA Memories : 레지스터, 공유 메모리, 글로벌 메모리"
date: 2024-05-31
draft: false
summary : "언제, 어떤 메모리를 사용해야 할까?"
tags: ["Multicore-GPU-Programming", "cuda"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 11
slug : "cuda-programming-matmul-1"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

지금까지 CUDA 커널 함수를 작성하는 방법과, CUDA 메모리 계층에 대해 공부했습니다.CUDA 커널 함수는 대량의 스레드에 의해 실행됩니다. 이 스레드에 의해 처리되는 데이터는 우선 호스트 메모리에서 디바이스 글로벌 메모리로 복사되어야 합니다. 그러나 이렇게 간단한 CUDA 커널은 하드웨어의 한계로 최대 퍼포먼스의 일부만 성능을 낼 수 있습니다. 이러한 성능 저하는 주로 글로벌 메모리 때문인데, 글로벌 메모리는 DRAM 으로 구현되어 있고, 긴 접근 레이턴시(주로 수백 클럭 사이클)를 가지고 있습니다. 데이터를 처리할 수 있는 수백개의 스레드가 있다고 해도, 호스트 메모리에서 디바이스 메모리로 데이터를 복사하는 과정에서 병목현상이 발생하면 모든 연산이 지연될 수 있습니다. 이러한 지연을 막기 위해서, CUDA는 글로벌 메모리에게 데이터 요청을 안해도 되는 방법들을 제공합니다. 따라서 본 포스팅에서는 이러한 **메모리를 사용해서 CUDA 커널의 효율성을 높이는 방법**에 대해 알아보겠습니다. 


특히, matmul 연산을 바탕으로 **공유 메모리(shared memory)** 와 **타일링** 에 대해 살펴보겠습니다.

## 1. Matrix Multiplication

멀티스레딩 환경에서 다루었던 [매트릭스 연산](https://ddoddii.github.io/post/cs/mgp/multithread-matmul/)을 다시 봅시다.

```c
for i = 1 to n
	for j = 1 to n
		for k = 1 to n
			P(i,j) = P(i,j) + M(i,k) * N(k,j)
```


<img width="359" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/983186cc-da7e-4b32-b879-ec61477c61c3">

$$P_{ij} =\Sigma_{k} \ M_{ik}N_{kj}$$

## 2. Importance of Memory Access Efficiency

메모리 접근의 효율성을 계산하기 위해, 앞서 봤던 matmul 계산 코드를 사용해봅시다. 

```cpp
__global__ void MatrixMulKernel(float* d_M, float* d_N, float* d_P, int width)
{
	// Calculate the row index of d_P element and d_M
	int row = blockIdx.y * blockDim.y + threadIdx.y;

	// Calculate the column index of d_P and d_N
	int col = blockIdx.x * blockDim.x + threadIdx.x;

	if ((row < width) && (col < width))
	{
		float Pvalue = 0;
		// each thread computes one element of the block sub-matrix
		for (int k = 0; k < width; ++k)
			Pvalue += d_M[row * width + k] * d_N[k * width + col];
		d_P[row * width + col] = Pvalue;
	}
}
```

커널의 실행 시간에서 가장 중요한 부분은 내부 내적을 연산하는 for loop 입니다.

```cpp
for (int k = 0; k < width; ++k)
	Pvalue += d_M[row * width + k] * d_N[k * width + col];
```

### CGMA ratio

이 루프의 각 이터레이션마다,  floating-point 곱셈과 floating-point 덧셈 별로 각각 2번의 글로벌 메모리 접근이 발생합니다. 첫번째 글로벌 메모리 접근은 `d_M[]` 원소를 가져오고, 두번째는 `d_N[]` 원소를 가져옵니다. 그 다음 두번의 floating-point 연산이 발생합니다. 따라서 **floating-point 연산과 글로벌 메모리 접근 비율**은 1:1 입니다. 이 비율을 **compute to global memory access (CGMA) ratio** 라고 합니다. 

CGMA 는 CUDA 커널의 성능에 관해 많은 것을 암시합니다. 요즘 디바이스에서, 글로벌 메모리 대역폭은 약 200BG/s 입니다. 각 4byte인 single-precision floating-point value 를 사용하면, 초당 50 (200/4) giga single-precision 피연산자만 로드할 수 있습니다. CGMA가 1이면, matmul 커널은 초당 50 giga floating-point operations(GFLOPS) 를 넘어서 실행할 수 없습니다. 50 GFLOPS 는 이론적으로 최대 성능인 1500 GFLOPS에 한참 못미치는 성능입니다. 따라서 커널의 성능을 높이기 위해서는 CGMA 비율을 더 올려야 합니다. matmul 코드가 최대 1500 GFLOPS 를 달성하려면, CGMA가 30이어야 합니다. 

## 3. CUDA Device Memory Types

CUDA는 프로그래머가 사용하여 CGMA를 높일 수 있는 **여러가지 종류의 메모리**를 제공합니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/061be07e-b87e-41f9-ae38-b1c9f7200366 'CUDA device memory model')

가장 아래 쪽에 **글로벌 메모리(global memory)** 와 **상수 메모리(constant memory)** 가 있습니다. 이 두개의 메모리는 호스트가 API 함수를 호출해서 write 또는 read 할 수 있습니다. 

**레지스터와 공유 메모리는 온-칩 메모리**입니다. 따라서 매우 빠른 속도로 접근할 수 있습니다. 커널 함수는 레지스터를 스레드가 자주 사용하는 스레드만의 로컬 변수를 저장하는 용도로 사용합니다. 레지스터는 스레드마다 할당됩니다. 공유 메모리는 스레드 블럭 단위로 할당되어서, 블럭 내 스레드들은 공유 메모리에 있는 변수에 접근할 수 있습니다. 공유 메모리는 인풋 데이터와 연산의 중간값을 저장하기에 좋은 장소입니다. 

레지스터, 공유 메모리, 글로벌 메모리가 CUDA 프로그래밍에서 실제로 어떻게 사용되는지 살펴봅시다. CUDA 프로그래밍 모델에서 **글로벌 메모리**는 폰 노이만 모델의 메모리에 해당합니다. 글로벌 메모리는 오프-칩 메모리로, DRAM 을 사용해 구현됩니다. DRAM은 긴 레이턴시와 상대적으로 작은 대역폭을 가집니다. **레지스터**는 폰 노이만 모델의 레지스터 파일에 해당합니다. 레지스터는 온-칩 메모리로, 작은 레이턴시와 높은 대역폭을 가집니다. 변수가 레지스터에 저장되면, 더 이상 메모리에 접근하지 않아도 되기 때문에 CGMA 의 향상에 큰 기여를 합니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/7147d689-7fee-4641-8216-92670beeedfa 'Memory vs. resgister based on von Neumann model')

레지스터에 접근하는 것은 글로벌 메모리에 접근하는 것보다 적은 명령어 개수를 사용합니다. 폰 노이만 모델에서, 프로세서는 PC 값을 사용해서 명령어를 메모리서부터 IR 로 가져옵니다. 이 명령어는 컴퓨터의 컴포넌트를 제어하기 위해 사용됩니다. 하나의 클럭 사이클에 가져와서 실행할 수 있는 명령어의 개수는 제한되어 있습니다. 따라서, 프로그램 실행에 더 많은 명령어가 필요하면, 프로그램을 실행하기 위해 더 오랜 시간이 걸립니다. 

현대 프로세서에서 대부분의 산수 명령어는 내장된 레지스터 오퍼랜드를 사용합니다. 예들 들어, floating 덧셈 명령어는 `fadd r1, r2, r3` 입니다. `r2` 와 `r3` 은 레지스터 파일 내에 오퍼랜드의 위치를 나타냅니다. 반면, 오퍼랜드 값이 글로벌 메모리에 있다면, 이 값을 다시 레지스터에 로드하는 명령어가 필요합니다. 따라서 이 로드하는 명령어 때문에 시간이 더 오래 걸립니다. 

```asm
load r2, r4, offset
fadd r1, r2, r3
```

오퍼랜드를 레지스터에 위치시키는 것을 선호하는 또 다른 이유가 있습니다. 현대 컴퓨터에서, 오퍼랜드를 레지스터에서 가져오는데 필요한 에너지는 오퍼랜드를 메모리에서 가져오는 에너지의 최소 몇 십 ~ 몇 백배 덜 필요합니다. 그러나 현대 GPU에서, 스레드에게 할당되는 레지스터의 수는 한정적이기 때문에 잘 사용해야 합니다. 

### Shared Memory vs. Registers

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1c3b95e0-95a2-4dea-b997-99eac3b12bec 'Shared memory vs. Registers in a CUDA device SM')

위는 CUDA 디바이스에서 **공유 메모리**와 **레지스터**를 보여줍니다. 둘 다 온-칩 메모리인 하지만, 기능과 비용에서 굉장히 다릅니다. 공유 메모리는 메모리의 일부로, 프로세서 칩에 있도록 설계되었습니다. **프로세서가 공유 메모리에 있는 데이터에 접근하기 위해서는 load 연산을 해야 합니다.** 하지만 공유 메모리는 온-칩이기 때문에 글로벌 메모리에서 로드해오는 것보다 훨씬 적은 레이턴시로 수행할 수 있습니다. 하지만 여전히 로드 명령어 때문에 공유 메모리는 레지스터보다 큰 레이턴시와 적은 대역폭을 가집니다. 

또 다른 차이점은, **공유 메모리에 있는 변수들은 스레드 블럭 내 모든 스레드들이 접근**할 수 있습니다. 반면 **레지스터에 있는 데이터는 스레드에게 프라이빗**합니다. 공유 메모리는 블럭 내 스레드들이 데이터를 효율적으로 공유하기 위해 설계되었습니다. CUDA 디바이스 SM은 여러 개의 프로세싱 유닛(SP)를 포함합니다. 블럭 내 스레드들은 이 프로세싱 유닛들에 분포하며 병렬적으로 연산을 수행합니다. 이때 공유 메모리를 사용해서 중간 연산 결과를 저장하거나, 인풋 데이터를 공유할 수 있습니다.

### Variable Declarations

이제 레지스터, 공유 메모리, 글로벌 메모리가 각각 다른 목적, 레이턴시, 대역폭을 가지는 것이 명확해졌습니다. 따라서 변수를 선언할 때 의도한 메모리에 있게 하는 것이 중요합니다. 아래의 표에서 각 변수 선언은 CUDA 변수에게 scope 와 lifetime 을 설정합니다. 

| Variable declaration                      | Memory   | Scope  | Lifetime    |
| ----------------------------------------- | -------- | ------ | ----------- |
| Automatic variables other than arrays     | register | thread | kernel      |
| Automatic array variables                 | local    | thread | kernel      |
| `__device__ __shared__` int SharedVar;    | shared   | block  | kernel      |
| `__device__` int GlobalVar;               | global   | grid   | application |
| `__device__ __constant__`int ConstantVar; | constant | grid   | application |

위의 matmul 코드에서, row, col, Pvalue 변수는 모두 automatic variables 로, 레지스터에 저장되고 스코프는 스레드인 변수입니다. 

변수가 `__shared__` 키워드로 선언되면, CUDA 에서 공유 메모리 변수입니다. 스코프는 스레드 블럭으로, 블럭 내의 모든 스레드들이 같은 값을 봅니다. 공유 메모리 변수의 라이프타임은 커널로, 커널이 실행을 멈추면 공유 메모리 변수도 사라집니다. 

변수가 `__constant__` 키워드로 선언되면, CUDA에서 상수 변수입니다. 이 변수는 함수 바깥에서 선언되어야 합니다. 스코프는 그리드로, 그리드 내에 있는 모든 스레드들이 같은 값을 봅니다. 상수 변수는 커널 함수에 인풋값을 저장하는데 사용됩니다.  상수 변수는 글로벌 메모리에 저장되지만, 효율적인 접근을 위해 캐시됩니다. 

변수가 `__device__` 키워드로 선언되면, 글로벌 변수로, 글로벌 메모리에 저장됩니다. 글로벌 변수에 대한 접근은 매우 느립니다. 글로벌 변수의 장점으로는 모든 커널 내 모든 스레드가 같은 값을 본다는 것입니다. 그리고 실행 내내 지속됩니다. 따라서 블럭 간에 정보를 공유할 때 사용됩니다. 하지만 글로벌 변수를 사용할 때 중요한 점은, 데이터 일관성을 보장하기 위한 동기화입니다. 

CUDA 에서 글로벌 메모리에 있는 데이터 객체를 가리키기 위해 포인터가 사용됩니다. 두가지 방법이 있는데, 첫번째로 객체가 호스트 함수에 의해 할당되었으면, 객체에 대한 포인터는 `cudaMalloc()` 함수에 의해 초기화되고 커널 함수에 파라미터로 넘겨집니다. 위의 코드에서 `d_M`, `d_M`, `d_P` 는 이러한 포인터의 예시입니다. 두번째는 글로벌 메모리에 의해 선언된 변수의 주소를 포인터 변수에 할당하는 것입니다. 예를 들어서, 커널 함수 내 `float* ptr = &GlobalVar;` 는 `GlobalVar` 의 주소를 포인터 변수 `ptr`에 할당합니다. 


## 4. A Strategy for reducing Global memory traffic : Tiling

CUDA 메모리 상에는 trade-off 가 존재합니다 : 글로벌 메모리는 크지만 느리고, 공유 메모리는 작지만 빠릅니다. 흔히 사용되는 전략은 데이터를 공유 메모리에 들어갈 수 있는 **tile** 로 나누는 것입니다. 중요한 것은 이러한 타일에 대한 연산은 각각에 대해 독립적으로 가능하다는 점입니다. 그러나 모든 자료구조가 타일로 파티션될 수 있는 것은 아닙니다.

앞서 매트릭스 연산에서, **타일링**을 사용할 수 있습니다. 아래에서, P 행렬에서 4개의 2 * 2  타일을 사용합니다. block(0,0) 에서 4개의 스레드가\\(P_{0,0},P_{0,1},P_{1,0},P_{1,1}\\) 을 연산합니다. 예를 들어, thread(0,0) 은 차례대로\\(M_{0,0},N_{0,0},M_{0,1},N_{1,0}, M_{0,2},N_{2,0},M_{0,3},N_{3,0}\\) 을 읽습니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/380bb662-eeeb-4666-abed-8cd6c084ba64 'Matrix Multiplication')

아래는 block(0,0) 에 있는 스레드들이 글로벌 메모리를 접근하는 것을 나타냅니다. 이때, thread(0,0), thread(0,1)  에서는 동시에 \\(M_{0,0}\\) 을 읽고 있는데 두개의 스레드가 협력해서 이 원소를 글로벌 메모리에서 한 번만 읽어올 수 있다면, 글로벌 메모리에 대한 접근을 절반으로 줄일 수 있습니다. 일반적으로 M,N 에 대한 모든 원소들을 2번씩 접근하는 것을 볼 수 있습니다. 따라서 4개의 스레드가 협력하면, 글로벌 메모리에 대한 접근을 절반으로 줄일 수 있습니다.

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/a740fe55-5af7-4e53-bdc7-0027b3476dbd 'Global memory access performed by threads in block(0,0)')

매트릭스 연산에서, 최대한 줄일 수 있는 글로벌 메모리 트래픽은 블럭의 차원에 비례합니다. 만약\\(N \times N\\) 블럭이 있다면, 줄일 수 있는 글로벌 메모리 트래픽은 N이 될 것입니다. 예를 들어,\\(16 \times 16\\) 블럭이면, 스레드 간의 협력을 통해 글로벌 메모리 트래픽을 1/16으로 줄일 수 있습니다. 

만약 스레드가 같은 패턴으로 글로벌 메모리에 접근한다면, 스레드 간 협력하기 수월합니다. 아래의 예시처럼, worker 끼리 비슷한 스케쥴을 가지고 있다면 교통체증을 유발하는 운전을 하지 않고 같이 카풀을 해서 교통체증을 줄일 수 있습니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/da21daa9-6790-49bb-b642-c4392bb88211 'Carpooling requires synchronization among people')

비슷하게 스레드가 DRAM 에 데이터를 요청할 때도, 스레드 간 데이터를 같은 패턴으로 접근하면 데이터를 여러 번 옮길 필요가 없습니다. 여러 개의 스레드가 같은 DRAM 위치에서 데이터를 요청하면, DRAM 은 데이터를 한 번만 전송하면 됩니다. 이렇게 스레드 간 타이밍을 맞추려면 **동기화**가 필요합니다. 뒤에서 barrier synchronization 을 사용해 스레드 간 타이밍을 맞추는 방법에 대해 보겠습니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1837c8a7-d356-4ebb-a980-91d3b5593d7d)




## 5. A Tiled Matrix - Matrix Multiplication Kernel

이제 스레드간 협력해서 글로벌 메모리의 접근을 절반으로 줄일 수 있는 알고리즘을 봅시다. 기본 아이디어는 스레드가 협력해서 M,N 원소들을 내적 연산에 사용하기 전에 공유 메모리에 미리 로드해놓는 것입니다. 그러나 공유 메모리의 크기는 매우 작아서, M,N 에 있는 모든 원소들을 로드할 수 없을 수 있습니다. 따라서 **M,N 행렬을 여러개의 작은 타일로 나누어서 공유 메모리에 로드**하면 됩니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/547deebf-2442-45db-99d2-36aeef5cc65e 'Tiling M,N matrixes to utilize shared memory')

위에서, M,N 행렬을\\(2 \times 2\\) 타일로 나누었습니다. 이제 스레드에 의한 내적 연산은 여러 단계로 나누어 집니다. 각 단계에서, 블럭 내의 스레드들은 협력해서 M 의 타일과 N의 타일을 공유 메모리로 로드합니다.

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1e7ed23a-4bbc-449f-953a-e50460c324d9 'Execution phases of a tiled matrix multiplication')

스레드들에 M,N 에서 각각 2개의 타일이 공유 메모리로 로드된 후에, 이 값들은 내적 연산을 위해 사용됩니다. 이때 공유 메모리에 있는 변수는 2번씩 사용됩니다. 예를 들어서, thread(1,1) 에 의해\\(Mds_{1,1}\\) 로 로드된\\(M_{1,1}\\) 값은 thread(0,1) 과 thread(1,1)에 의해 한 번씩 사용됩니다. 

내적 연산은 2 단계에서 일어납니다. 각 단계에서, 내적 연산의 결과는 `PValue` 값에 더해집니다. `PValue`는 automatic variable 로, 스레드 마다 생성됩니다. 만약 인풋 행렬의 차원이 N 이고 타일의 사이즈가 TILE_WIDTH 이면, 내적 연산은 N/TILE_WIDTH 단계로 수행됩니다. 

이렇게 타일링 방법으로 작성한 코드는 아래와 같습니다. 

```cpp
#define TILE_WIDTH 16

__global__ void MatrixMulKernel(float* d_M, float* d_N, float* d_P, int width)
{
	__shared__ float Mds[TILE_WIDTH][TILE_WIDTH];
	__shared__ float Nds[TILE_WIDTH][TILE_WIDTH];

	// automatic variables (stored in register, per thread)
	int bx = blockIdx.x; int by = blockIdx.y;
	int tx = threadIdx.x; int ty = threadIdx.y;
	
	// Calculate the row index of d_P element and d_M
	int row = by * TILE_WIDTH + ty;
	int col = bx * TILE_WIDTH + tx;

	float Pvalue = 0;
	// Loop over the d_M and d_N tiles required to compute d_P element
	for (int m = 0; m < width/TILE_WIDTH; ++m) {
		// Collaborative loading of d_M and d_N tiles into shared memory
		Mds[tx][ty] = d_M[row * width + m * TILE_WIDTH + tx];
		Nds[ty][tx] = d_N[(m * TILE_WIDTH + ty) * width + col];
		__syncthreads(); // Wait for copy to finish

		for (int k = 0; k < TILE_WIDTH; ++k) {
			Pvalue += Mds[ty][k] * Nds[k][tx];
		}
		__syncthreads(); // Wait for compute to complete
	}
	d_P[row * width + col] = Pvalue;
}
```



![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/8ad00fd1-fb76-4eba-b02b-16077217d5d2 'Calculation of matrix indices in tiled multiplication')

공유 메모리에 있는 데이터로 연산을 수행하기 전에, 행렬M,N이 모두 복사되었는지 확인하는 단계를 거쳐야 합니다. 즉, 모든 스레드가 작업 완료를 했는지 확인해야 합니다. 블록 내 스레드들을 동기화시키는 함수가 `__syncthreads()` 입니다. `__syncthreads()`는 마치 스레드 블록 내의 barrier 처럼 작동합니다. 모든 스레드들이 이 장벽에 도착할 때까지 기다립니다.

<img width="276" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/96201869-1d7d-454d-8c0b-371b358a3da6">


## 6. Memory as a Limiting Factor to Parallelism

CUDA 레지스터와 공유 메모리가 글로벌 메모리에 대한 접근 횟수를 크게 줄여주지만, 이 용량을 초과하지 않도록 조심해야 합니다. 각 CUDA 디바이스는 스레드 실행에 제한된 리소스를 제공합니다. 

예시 디바이스 D의 상황을 봅시다. 최대 1536개 스레드, 최대 16384 레지스터를 가질 수 있으면 스레드 당 10개의 레지스터만 사용할 수 있습니다. 

SM이 사용가능한 레지스터 개수는 디바이스마다 다릅니다. 어플리케이션은 동적으로 SM에서 사용가능한 레지스터 수를 결정하고, 적절한 버전의 커널을 결정할 수 있습니다. 이것은 `cudaGetDeviceProperties()` 함수를 사용해서 할 수 있습니다. 

공유 메모리도 SM에 할당된 스레드의 개수를 제한할 수 있습니다. 디바이스 D가 SM 당 16384(16K) bytes 의 공유 메모리를 가지고 있다고 합시다. 공유 메모리는 블럭 단위로 사용됩니다. SM이 8개 블럭까지 수용할 수 있으면, 각 블럭은 2K 이상의 공유 메모리를 사용하면 안 됩니다. 만약 각 블럭이 2K 이상의 공유 메모리를 사용하면, SM이 수용할 수 있는 블럭의 수가 줄어듭니다. 

앞에서 본 매트릭스 연산에서, 공유 메모리가 제한 요소로 작동할 수 있습니다.\\(16 \times 16\\) 타일 사이즈에서, 각 블럭은\\(16 \times 16 \times 4 = 1K\\) 의 Mds 가 필요합니다. Nds 도 마찬가지로 1KB 가 필요합니다. 따라서 각 블럭은 2K의 공유 메모리를 사용합니다. 따라서 16K의 공유 메모리는 블럭 개수를 8개로 제한합니다. 만약 768개의 스레드 만 SM안에서 수용할 수 있으면, SM 내부의 블럭 개수는 3개로 제한됩니다. 따라서\\(3 \times 2KB = 6KB\\) 의 공유 메모리만 사용될 것입니다. 

### Dynamic shared memory allocation

위의 코드에서, TILE_WIDTH를 수정하려면 직접 `#define TILE_WIDTH` 를 수정해야 했습니다. 컴파일 시점에 Mds, Nds의 사이즈를 결정할 수 있게 하려면, `extern` 키워드를 사용하면 됩니다. 이 키워드를 사용하면 선언할 때 배열의 크기를 생략할 수 있습니다.

```cpp
extern __shared__ Mds[];
extern __shared__ Nds[];
```

커널을 런칭할 때, 디바이스 쿼리 결과에 따라 얼만큼의 공유 메모리를 사용할 지 결정할 수 있습니다. 

```cpp
size_t size = calculate_appropriate_SM_usage(dev_prop.sharedMemPerBlock, ...);
matrixMulKernel <<<dimGrid, dimBlock, size>>>(Md,Nd,Pd,width);
```




## Reference
- Programming Massively Parallel Processors, ch.5
- https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/#device-memory-spaces

- Multicore and GPU Programming, 연세대학교 박영준 교수님