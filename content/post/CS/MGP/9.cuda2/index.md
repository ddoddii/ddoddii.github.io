---
title: "CPU-GPU 통신 및 CUDA를 활용한 이미지 프로세싱 기법"
date: 2024-05-28
draft: false
summary : "CUDA 프로그래밍 작성에 필수적인 CUDA 스레드 계층구조와 스레드 인덱싱"
tags: ["Multicore-GPU-Programming", "cuda"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 9
slug : "cuda-programming-2"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

본 포스팅에서는 CUDA 스레드가 무엇인지와, 스레드 계층구조에 대해 살펴보겠습니다. 

## CPU와 GPU간 관계

<img width="857" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2ee6a0d8-9677-4cd4-ae9d-06c311908c28">


**CPU**는 100개 이해의 스레드를 가능한 빠르게 실행하도록 설계되었습니다. 큰 캐시가 있고, 컨텍스트 스위칭 비용으로 인해 코어 당 몇 개 수준의 스레드가 실행되는 것이 유리합니다. 성능을 위해 캐시와 프리페칭(prefetching)에 의존합니다. CPU는 브랜치가 많고, 랜덤 패턴 접근이 많은 복잡한 연산에 유리합니다.

<img width="826" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/16d31fa8-1de1-4774-a2b9-53a601ed58cd">

**GPU**는 수천개의 스레드를 병렬로 실행하도록 설계되었습니다. 작은 사이즈의 캐시를 가지고 있고, 메모리 밴드위스가 큽니다. 성능을 위해 멀티 스레딩에 의존합니다. GPU는 대량의 데이터-병렬 연산 어플리케이션에 유리합니다.

||CPU|GPU
|---|---|---|
|발전 방향|각 코어의 성능 향상|병렬 처리(코어 수)증가|
|코어 수|1~수십 개|수백~수천 개|
|개별 코어 성능| 높음| 낮음|
|구조|SSID, MIMD| SIMT|
|공간 분배| 캐시 및 제어 유닛에 많은 공간 할당| 연산 유닛에 많은 공간 할당|
|메모리 크기| 수십 GB ~ 1TB| ~수십 GB|
|메모리 접근| 메모리 접근 지연 시간을 최적화| 높은 메모리 대역폭 확보에 집중|
|장점| 불규칙한 작업 흐름 및 메모리 접근 패턴에 잘 대응함| FLOPS 관점에서 CPU 대비 월등한 처리 성능|
|단점| FLOPS 관점에서 GPU 대비 낮은 성능| 불규칙한 작업 흐름 및 메모리 접근에 의한 성능 저하 발생|

### CPU-GPU 데이터 전송 오버헤드

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1115c030-f08e-465b-af89-20bc2d4b5b33)

CPU와 GPU는 각각의 메모리를 가지고 있는데, CPU는 **호스트 메모리(Host Memory)** 라고 하고, GPU는 **디바이스 메모리(Device Memory)** 라고 합니다. 이때 CPU에서 바로 Device 메모리로 접근하는 것은 불가하고, 호스트 메모리에서 데이터 전송을 해야 합니다. 이것을 PCI-e(PCI express Interface) 또는 NVLINK 라고 합니다. 메모리 간 데이터를 아주 많이 전송해야 하기 때문에 이 속도를 높이는 것이 성능에 아주 중요한 요소입니다.


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dbc4ff14-b86a-4af4-90a6-2bef771313d3)

위에서 보다시피 Control Intensive 연산에는 CPU를 사용하고, Compute Intensive 연산에는 GPU를 사용하는 것이 유리합니다. 이때 중간에 스위칭 될때는 데이터를 전송해야 합니다. 만약 데이터 전송 오버헤드가 GPU에서 연산을 하는 이점보다 크다면, 그냥 CPU에서 계속 연산을 하는 것이 유리할 수 도 있습니다. 따라서 이러한 **데이터 전송 오버헤드**를 꼭 고려해야 합니다.

### 데이터를 어디에 저장할 것인가?

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4eb08b53-22fd-4718-9e94-67ad0c8d4411)

위의 상황처럼 GPU의 디바이스 메모리에 있는 데이터를 CPU에서 사용하고 싶을 때, 데이터를 어디에 저장해야 할까요? 정답은 **호스트 메모리, 디바이스 메모리 둘 다** 입니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/42ef58c7-09f5-41ca-bb5b-e4023d718d38)

[CUDA 프로그래밍 기초 포스팅](https://ddoddii.github.io/post/cs/mgp/cuda-programming/#cuda-%EA%B8%B0%EC%B4%88-%EB%A9%94%EB%AA%A8%EB%A6%AC-api)의 CUDA 기초 메모리 API에서 잠시 다룬 `Memcpy()` 를 사용해서 데이터를 복사해야 합니다.

<img width="611" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b66e5ae2-d633-46b2-bbc9-f56e6628c1da">

## Vector Addition

CUDA를 사용해서 **벡터의 합을 구하는 연산**을 작성해봅시다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/13908988-bf70-4a1e-bfb9-2e237622e1b0)

```cpp
float *A_h, *B_h, *C_h; //host(CPU)
float *A_d, *B_d, *C_d; // device(GPU)

cudaMalloc((void**) &A_d, size);
cudaMemcpy(A_d,A_h,size,cudaMemcpyHostToDevice);
cudaMalloc((void**) &B_d, size);
cudaMemcpy(B_d,B_h,size,cudaMemcpyHostToDevice);
cudaMalloc((void**) &C_d, size);

// kernel invocation
vecAddKernel<<<ceil(N/256.0), 256>>>(A_d,B_d,C_d,N);

// Transfer C from device to host
cudaMemcpy(C_h,C_d,size,cudaMemcpyDeviceToHost);

// Free CUDA memory
cudaFree(A_d);
cudaFree(B_d);
cudaFree(C_d);
```

GPU에서는 CPU의 스레드보다 **훨씬 작은 개념의 스레드**를 사용합니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/54c87ef0-bc1f-4f20-86ce-1999ce653b6a)

CUDA에서 벡터 합을 연산하는 코드는 아래와 같이 작성할 수 있습니다.

```cpp
__global__
void vecAddKernel(float* A_d, float* B_d, float* C_d, int n)
{
    int i = threadIdx.x + blockDim.x * blockIdx.x;
    if (i<n)
        C_d[i] = A_d[i] + B_d[i];
}

int main()
{
    ...
    vecAddKernel <<<ceil(N/256.0), 256>>>(A_d,B_d,C_d,N);
    ...
}
```

이 코드를 본격적으로 이해하려면 CUDA 스레드 계층 구조에 대해 알아야 합니다.

## CUDA 스레드 계층 구조

**CUDA의 스레드 계층(CUDA thread hierarchy)** 는 **스레드, 워프, 블록, 그리드**라는 4개의 계층으로 이루어져 있습니다. 

<img width="980" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/081792ec-988f-4153-ba5b-a7c6bf1c17dd">

- **스레드**

    CUDA 스레드 계층 구조에서 가장 작은 단위는 스레드입니다. 스레드는 **CUDA에서 연산을 수행하거나 CUDA 코어를 사용하는 기본 단위**입니다. 작성한 커널 코드는 모든 스레드에 공유되며 각 스레드가 독립적으로 커널 코드를 수행합니다. 

- **워프**

    워프는 **32개의 스레드를 하나로 묶은 것**이며, CUDA의 기본 수행 단위입니다. 기본 수행 단위라 하는 것은, 한 워프에 속한 스레드들은 하나의 제어 장치에 의해 제어된다는 것입니다. GPU의 SIMT 구조에서 멀티 스레드의 단위가 되는 것이 워프입니다. 하나의 명령에 따라 32개의 스레드가 동시에 움직입니다. 

- **블록**

    블록(=스레드 블록)은 **워프들의 집합**입니다. 블록이 가지는 중요한 의미는 스레드에 부여되는 고유 번호(ID)에 있습니다. 하나의 블록에 포함된 각 스레드는 자신만의 고유한 스레드 번호(thread ID)를 갖습니다. 즉, 하나의 블록 내에서는 동일한 번호를 갖는 스레드는 없습니다. 반면, 다른 블록에 속하는 스레드들은 같은 스레드 번호를 가질 수 있습니다. 각 블록은 자신만의 고유한 블록 번호(block ID)가 있으며, 우리가 원하는 스레드를 지칭하기 위해서는 블록 번호와 스레드 번호 모두 사용해야 합니다. 

    블록은 1차원, 2차원 또는 3차원 형태로 배치될 수 있으며 스레드 번호도 배치에 따라 최대 3차원 번호를 가질 수 있습니다. 

- **그리드**

    그리드는 CUDA 스레드 계층 구조에서 가장 상위 단계입니다. 그리드는 여러 개의 블록을 포함하는 블록들의 그룹입니다. 하나의 그리드에 포함된 블록들은 서로 다른 자신만의 고유한 블록 번호(block ID)를 갖습니다. 블록과 마찬가지로 그리드 내 블록 또한 1차원, 2차원 또는 3차원 형태로 배치될 수 있습니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/12c4c15e-3ce4-428f-a659-c9bde59c33f1)

### CUDA 스레드 계층을 위한 내장 변수들

스레드들이 자신이 처리할 데이터가 무엇인지 알기 위해서는 자신이 어느 블록에 속해있는지, 블록 내 자신의 스레드 번호를 알아야 합니다. 이를 위해서 CUDA에서는 현재 그리드 및 블록의 형태와 각 스레드가 자신이 속한 블록 번호, 그리고 자신의 스레드 번호를 확인할 수 있는 내장 변수를 제공합니다.

<img width="606" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/99f32212-5bb7-4325-a7ab-c1a6ce59285f">

- `gridDim`

    `gridDim`은 그리드의 형태 정보를 담고 있는 구조체형 내장 변수로, x,y,z 멤버가 각각 차원의 크기를 담고 있습니다. 위의 사진에서 (3,2,1) 그리드이면, gridDim.x = 3, gridDim.y = 2, gridDim.z = 1 입니다. 실제로 사용하지 않는 차원의 크기는 1로 표시됩니다. gridDim은 커널 내 모든 스레드가 공유하는 내장 변수 입니다.  

- `blockIdx`

    `blockIdx`는 현재 스레드가 속한 블록의 번호를 담고 있는 구조체형 내장 변수입니다. 위의 사진에서는 6개의 블럭을 포함하고 있고, 2차원 그리드이기에 각 블록은 2차원 블록 번호를 가집니다. 블록 번호는 0부터 시작하며, 오른쪽으로 갈수록 x-차원 번호가 (0,0), (1,0), (2,0) 으로 1씩 증가합니다. blockIdx 변수는 한 블록에 속한 스레드들이 모두 공유합니다. 

- `blockDim`

    `blockDim`은 블록의 형태 정보를 담고 있는 구조체형 내장 변수입니다. 커널이 실행될 때 그리드 및 블록의 형태가 결정되며, 한 그리드 내 모든 블록은 동일한 형태를 가집니다. blockDim은 그리드 내 모든 스레드가 공유합니다.

- `threadIdx`
  
  `threadIdx`는 현재 스레드가 부여받은 스레드 번호를 담고 있는 구조체형 내장 변수입니다. 한 블록 내 스레드들은 서로 다른 스레드 번호를 가지며 블록 형태에 따라 최대 3차원의 번호를 가질 수 있습니다. 

- **스레드 번호와 워프의 구성**

    워프는 연속된 32개의 스레드로 구성됩니다. 스레드의 연속성은 threadIdx의 x-차원, y-차원, z-차원 순으로 결정됩니다. 즉 (0,0,0) ~ (31,0,0) 의 스레드가 하나의 워프를 구성합니다. 


<img width="650" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/61e8b40c-f033-48c0-99d8-9228ce02fa8b">


### CUDA 스레드 구조와 커널 호출

스레드 레이아웃(thread layout)은 스레드의 배치 형태를 지칭하는 것으로, 그리드와 블록의 형태로 정의됩니다. 스레드 레이아웃는 커널 호출 시 설정하며, 우리가 커널을 호출할 때 사용한 `<<<...>>>` 문법을 통해 전달합니다.

```cpp
Kernel <<<그리드 형태, 블록 형태>>>()
```

만약 `<<<1,n>>>` 으로 지정하면, (1,1,1) 크기의 그리드를 사용하고, (n,1,1) 블록의 크기를 사용하라는 의미입니다. 

<img width="541" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e53b4615-2cd0-4643-b407-b32cdc672daf">


앞서 봤던 벡터 합 연산 코드를 다시 봅시다. 이 경우에는 그리드는 1차원으로, n/256개의 블럭을 가지고 있고 블럭도 1차원으로 256개의 스레드를 가지고 있습니다.


```cpp
int main 
{
    dim3 DimGrid(n/256, 1, 1); // x,y,z
    if (n % 256)
        DimGrid.x++;
    dim3 DimBlock(256, 1, 1);

    vecAddKernel<<<DimGrid, DimBlock>>>(A_d,B_d,C_d,N)
}
```

### 스레드 인덱싱

스레드 인덱싱이 중요한 이유는 2차원, 3차원 배열이라고 해도 실제 메모리에 저장되는 형태는 1차원 배열이기 때문입니다. 저장 규칙은 낮은 차원(x-차원) 부터 높은 차원(y-차원) 순서로 저장합니다. 스레드의 전역 번호(global ID) 를 구하는 과정을 단계적으로 알아봅시다.

1. **블록 내의 스레드 전역 번호**

- **1차원 블록**
        
    1D 블록의 경우 threadIdx.x 가 스레드의 전역 번호와 같습니다. 

- **2차원 블록** 

    <img width="498" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cedd530d-6452-4e0c-82e9-7dae12e89bd4">

    2D 블록의 경우, 1차원 형태의 하위 블록 여러 개가 y-차원으로 나열되어 있습니다. 이때 필요한 정보는, 자신이 속한 1차원 하위 블럭 앞 까지 스레드 개수(blockDim.x * threadIdx.y) 와 자신이 속한 1차원 하위 블럭 안에서 자신의 스레드 번호(threadIdx.x) 입니다.

    ```
    2D_BLOCK_TID = (blockDim.x * threadIdx.y) + threadIdx.x
    ```

- **3차원 블록**

    <img width="649" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/9e2643b2-ca2f-4bd9-aa19-2dd7c76a2254">


    3D 블록의 경우, 2차원 형태의 하위 블록 여러 개가 z-차원으로 나열되어 있습니다. 이때 필요한 정보는, 자신이 속한 2차원 하위 블럭 앞 까지 스레드 개수(blockDim.x * blockDim.y * threadIdx.z) 와 자신이 속한 2차원 하위 블럭 안에서 자신의 스레드 번호(2D_BLOCK_TID) 입니다.

    ```
    3D_BLOCK_TID = (blockDim.x * blockDim.y * threadIdx.z) + 2D_BLOCK_TID
    ```

1. **그리드 내 스레드 전역 번호**

그리그 내 블록이 1개라면 `TID_IN_BLOCK`이 그리드 내에서 스레드의 전역 번호가 되지만, 블럭이 여러 개라면 블럭 번호도 고려해야 합니다. 이때 고려해야 하는 정보는 자신이 속한 블록 앞 블록까지 스레드 개수와 자신의 속한 블록 내에서 자신이 몇번째 스레드인지(`TID_IN_BLOCK`) 입니다. 

- **1차원 그리드**

    고려해야 하는 정보는, 블록 하나에 속한 스레드 개수(NUM_THREAD_IN_BLOCK = blockDim.z * blockDim.y * blockDim.x) 와 자신이 속한 블록의 번호 입니다.

    ```
    1D_GRID_TID = (blockIdx.x * (NUM_THREAD_IN_BLOCK)) + TID_IN_BLOCK
    ```

- **2차원 그리드**

    2차원 그리드는 1차원 형태의 하위 그리드 여러 개가 y-차원으로 나열된 형태입니다. 필요한 정보는 1차원 하위 그리드 안의 스레드 개수(blockIdx.x * (NUM_THREAD_IN_BLOCK)), 자신이 속한 1차원 하위 그리드 번호(blockIdx.y), 자신이 속한 1차원 하위 그리드 내에서 자신의 스레드 번호(1D_GRID_TID) 입니다. 

    ```
    2D_GRID_TID = blockIdx.y * (gridDim.x * (NUM_THREAD_IN_BLOCK)) + 1D_GRID_TID
    ```
     

- **3차원 그리드**

    필요한 정보는 2차원 하위 그리드 안의 스레드 개수(gridDim.x * gridDim.y * (NUM_THREAD_IN_BLOCK)), 자신이 속한 2차원 하위 그리드 번호(blockIdx.z), 자신이 속한 2차원 하위 그리드 내에서 자신의 스레드 번호(2D_GRID_TID) 입니다. 

    ```
    3D_GRID_TID = blockIdx.z * (gridDim.x * gridDim.y * (NUM_THREAD_IN_BLOCK)) + 2D_GRID_TID
    ```

#### 1D grid, 2D blocks

1차원 그리드, 2차원 블럭의 경우에 글로벌 스레드 인덱스는 아래와 같이 계산할 수 있습니다.

```cpp
__device__
int getGlobalIdx_1D_2D() 
{
    return blockIdx.x * blockDim.x * blockDim.y 
    + threadIdx.y * blockDim.x + threadIdx.x;
}
```

 `blockIdx.x * blockDim.x * blockDim.y` 는 현재 블럭 이전에 있는 모든 스레드의 개수를 계산합니다. 각 블럭은 `blockDim.x * blockDim.y` 만큼의 스레드를 가지고 있습니다. 이것을 `blockIdx.x` 와 곱해서 현재 블럭 이전의 스레드 개수를 계산할 수 있습니다.

`threadIdx.y * blockDim.x + threadIdx.x` 는 현재 블럭 내에서 스레드의 인덱스를 계산합니다. 

예를 들어 DimGrid 이 (3,1,1) 이고, DimBlock 이 (4,4,1),  blockIdx.x = 2 이면 그리드 내에는 3개의 블럭이 있고, 각 블럭은 4*4 = 16개의 스레드가 있습니다. 따라서 그리드 내 총 스레드 개수는 48개 입니다. 

특정 스레드(blockIdx.x = 2, threadIdx.x = 3, threadIdx.y = 2)의 글로벌 인덱스를 계산해봅시다. 첫번째로 블럭 오프셋은, 
$$Block \ Offset = blockIdx.x \times (blockDim.x \times blockDim.y)$$ 

로 계산할 수 있습니다. 이 경우에는 2 * 4 * 4 = 32 입니다. 블럭 내에서 오프셋을 계산하면, 
$$Within-Block \ Offset = threadIdx.y \times blockDim.x + threadIdx.x$$
이므로 2*4 + 3 = 11 입니다. 따라서 글로벌 인덱스는 32 + 11 = 43이 됩니다. 

![IMG_3BDE7F115578-1](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/67beb168-f33d-4c36-91c3-3178825adbf8)



#### 2D grid, 3D blocks

2차원 그리드, 3차원 블럭의 경우에 글로벌 스레드 인덱스는 아래와 같이 계산할 수 있습니다.

```cpp
__device__
int getGlobalIdx_2D_3D()
{
    int blockId = blockIdx.x + blockIdx.y * gridDim.x;
    int threadId = blockId * (blockDim.x * blockDim.y * blockDim.z)
    + (threadIdx.z * (blockDim.x * blockDim.y))
    + (threadIdx.y * blockDim.x)
    + threadIdx.x;

    return threadId;
}
```

![IMG_012C4A554BDB-1](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d4d28787-fcb0-4aa8-9941-78e88b9dd233)




## 이미지 프로세싱

### 흑백 이미지로 변환

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/34514046-0d81-4dc5-a45c-9bed022551a3)

이미지 프로세싱은 각 픽셀에 대해 연산을 수행하며, GPU를 활용하는 대표적인 분야입니다. 이미지를 흑백으로 만드는 연산은 각 픽셀에 대해 아래의 연산을 수행하면 됩니다.

$$L = 0.21 * r + 0.71 * g + 0.07 * b$$

이때 각 픽셀에 대한 연산은 독립적으로 수행될 수 있습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/80f66912-ea03-418a-a88a-87769aa0663c)

흑백 변환 연산을 수행하는 코드를 작성해봅시다. 이때 76(width) * 62(height) 사진을 16 * 16 블럭으로 커버한다고 가정해 봅시다. 이때 5*4개의 블럭을 사용한다면, 오른쪽 엣지에서 처음 12개의 스레드 (64~76)만 사용하고, 아래 엣지에서는 처음 14개 스레드(48~64)만 사용합니다. 이때 `(Col < width && Row < height)` 를 통해 현재 스레드가 사진의 범위에 있는지 확인해야 allocation fault 를 막을 수 있습니다. 이 코너 케이스를 확인하지 않는다면 GPU의 낭비가 발생할 수 있습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/25dbdf35-1813-4a1e-a75a-3dada6fc845f)


```cpp
const int CHANNELS = 3;
__global__
void colorToGreyScaleConversion(unsigned char* rbgImage, unsigned char* grayImage, int width, int height)
{
    int Col = threadIdx.x + blockIdx.x * blockDim.x;
    int Row = threadIdx.y + blockIdx.y * blockDim.y;

    if (Col < width && Row < height) // To prevent allocation fault !! 
    {
        // get 1D coordinate for grayscale image
        int grayOffset = Row * width + Col;
        // one can think of the RGB image having
        // CHANNEL times columns of the gray scale image
        int rgbOffset = grayOffset * CHANNELS;
        unsigned char r = rgbImage[rgbOffset]; // red value for pixel
        unsigned char g = rgbImage[rgbOffset + 1]; // green value for pixel
        unsigned char b = rgbImage[rgbOffset + 2]; // blue value for pixel
        // perform the rescaling and store it
        // We multiply by floating point constants
        grayImage[grayOffset] = 0.21f*r + 0.71f*g + 0.07f*b;
    }
}
```

### 이미지 블러 처리

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d3cfbcb6-1a9e-4dbb-aca9-ff561d199656)

이미지 블러 처리는 어떻게 할까요? 블러 처리는 픽셀 사이의 경계를 흐리게 하면 됩니다. 즉 각 픽셀 간의 average 를 계산하면 됩니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bfdc8182-a615-4a1c-b6a9-5ae15c7a5bb2)

이미지 블러 처리도 코드로 작성해 봅시다. 

```cpp
__global__
void blurKernel(unsigned char* in, unsigned char* out, int w, int h)
{
    int Col = threadIdx.x + blockIdx.x * blockDim.x;
    int Row = threadIdx.y + blockIdx.y * blockDim.y;
    if (Col < w && Row < h)
    {
        int pixVal = 0;
        int pixels = 0;
        // Get the average of the surrounding BLUR_SIZE * BLUR_SIZE box
        for (int blurRow = -BLUR_SIZE; blurRow < BLUR_SIZE + 1; ++blurRow)
        {
            for (int blurCol = -BLUR_SIZE; blurCol < BLUR_SIZE + 1; ++blurCol)
            {
                int curRow = Row + blurRow;
                int curCol = Col + blurCol;
                // Verify we have a valid image pixel
                if (curRow > -1 && curRow < h && curCol > -1 && curCol < w)
                {
                    pixVal += in[curRow * w + curCol]; // Boundary handling
                    pixels++; // Keep track of number of pixels in the avg
                }
            }
        }
        // Write our new pixel value out
        out[Row * w + Col] = (unsigned char)(pixVal/pixels);
    }
}

```

이때 바운더리를 핸들링 하는 것이 중요합니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ab0f1c52-ea8e-474d-9748-30fea2477108)


다음 포스팅에서는 **Nvidia GPU의 메모리 구조**에 대해 살펴보고, 하드웨어 구조에 따라서 성능이 어떻게 바뀌는지에 대해 살펴보겠습니다.



## Reference
- https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#thread-hierarchy
- https://kdm.icm.edu.pl/Tutorials/GPU-intro/introduction.en/
- https://www.eecs.umich.edu/courses/eecs498-APP/resources/materials/CUDA-Thread-Indexing-Cheatsheet.pdf
- CUDA 기반 GPU 병렬 처리 프로그래밍, ch.4 김덕수 저
- Multicore and GPU Programming, 연세대학교 박영준 교수님