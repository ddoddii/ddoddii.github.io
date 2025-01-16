---
title: "CUDA 기반 matmul 성능 최적화하기"
date: 2024-06-08
draft: false
summary : "CUDA stream 사용하기"
tags: ["Multicore-GPU-Programming", "cuda"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 12
slug : "cuda-programming-matmul-2"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}


## Optimizing Memory Access


![img](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/501e2be5-c53d-4706-89e6-9b529a122d0b '캐시를 통한 전역 메모리 접근 흐름')

**L2 캐시**에 대한 기본 전송 흐름은 **32byte** 입니다. 즉, 한번의 메모리 전송으로 최대 연속된 32byte의 데이터를 읽거나 쓸 수 있습니다. **L1 캐시**의 캐시 라인 크기는 **128byte** 입니다. 각 캐시 라인은 디바이스 메모리에서 크기가 같은 메모리 블럭에 대응됩니다. 캐시 라인이 128byte 인 L1 캐시를 예로 들면, 128byte 단위로 나누어진 디바이스 메모리 블록과 대응됩니다. 만약 각 스레드가 int 변수와 같은 4byte 데이터에 접근한다면, 한 워프에서 접근해야 하는 총 데이터 크기는 128byte가 됩니다. 그리고 워프 내 스레드들의 메모리 접근 요청 위치가 연속된 메모리 주소(메모리 공간)이면서 캐시 라인과 정확하게 그 위치가 일치한다면 이 요청은 한 번의 L1캐시 전송으로 처리됩니다. 하지만 이러한 조건이 맞지 않으면, 워프의 데이터 요청을 처리하기 위해 메모리 전송 횟수가 증가합니다.

### Aligned memory acess & Coalesced memory access

글로벌 메모리 접근을 최적화하기 위해 기억해야 할 것은 정렬된(aligned) 메모리 접근과 병합된(coalesced) 메모리 접근입니다. **정렬된 메모리 접근**이란 요청한 데이터의 시작점이 캐시에 대응하는 데이터 블록의 시작 지점과 일치하는 경우입니다. **병합된 메모리 접근**은 32개의 스레드가 연속적인 데이터에 접근하는 것 입니다. 

1. 정렬 & 병합된 메모리 접근

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/3906760f-1ba9-405e-b940-20e43e0bf7c9 '정렬 & 병합된 메모리 접근')

가장 이상적인 정렬 & 병합된 메모리 접근 방식입니다. 워프 내 32개 스레드가 시작 주소가 128이고 연속된 128byte에 접근하는 경우로, 캐시 라인의 크기가 128byte 인 L1 캐시에 대한 데이터 전송은 한 번의 메모리 전송으로 수행할 수 있습니다. 캐시 라인 크기가 32byte인 L2 캐시의 경우, 4번의 메모리 전송으로 요청된 메모리 접근을 처리할 수 있습니다. 

2. 정렬 X & 병합된 메모리 접근

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1f0d4b52-b858-4b43-93e5-56264d0f5a2a '정렬되지 않은 병합된 메모리 접근')

이 경우 접근하는 메모리 영역은 연속적이지만, 시작 메모리 주소가 128의 배수가 아닙니다. 따라서 L1캐시라면 2번, L2캐시라면 5번의 메모리 전송을 필요로 합니다. 

3. 정렬 X & 병합 X 인 메모리 접근

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/4a25e77c-490e-4cec-b398-06aa9ac63397 '정렬되지 않고 병합되지 않은 메모리 접근')

이 경우 워프 내 스레드의 메모리 접근 패턴이 불규칙하고, 여러 캐시 라인에 접근해야 합니다. 만약 워프 내 모든 스레드가 서로 다른 캐시 라인에 접근한다면 최대 32번의 메모리 전송을 요구할 수도 있습니다. 즉, 이상적인 경우보다 최대 32배의 메모리 전송 부하가 발생할 수 있습니다.

### Matmul and Coalescing


<img width="750" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/389d5a0e-c8a6-4113-a7de-e8a4e7b94e11">

2가지 방법 중, 어느 것이 더 빠를까요? 정답은 N의 한 칼럼마다 스레드를 할당하는 방식입니다. 

1. **열마다 스레드 할당(Coalescing)**

<img width="492" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/399064a4-7195-4eb1-a494-17943310de9d">

<img width="872" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/72665bfc-9812-4697-a05e-a358189cc609">

워프 내에 있는 스레드가 같은 캐시 라인에 접근하면, 메모리에 대한 접근은 연속적으로 이루어집니다. 따라서 병합된 메모리 접근이며, 메모리에 한 번만 접근하면 됩니다. 

2. **행마다 스레드 할당(No Coalescing)**

<img width="347" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/fd778b77-739d-4281-a7f0-9f7c853adcab">


<img width="1040" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1586dafc-f26d-4c32-beb7-f9c1c493a8cb">


이때는 여러 번(4번)의 메모리 접근을 해야 합니다. 따라서 병합된 메모리 접근을 할 수 있도록 코드를 작성해야 합니다. 

### Dynamic shared memory allocation

```cpp
__global__ void MatrixMulKernel(float* M, float* N, float* P, int Width)
{
    // Declare shared memory variable
    __shared__ float subTileM[TILE_WIDTH][TILE_WIDTH];
    __shared__ float subTileN[TILE_WIDTH][TILE_WIDTH];
    ...
}
```

우리는 지금까지 정적 공유 메모리 할당을 했습니다. 즉, 하나의 스레드 당 하나의 원소를 할당했습니다. 만약 우리가 아래와 같이 서로 다른 블럭 사이즈로 커널을 여러 번 런칭하고 싶으면 어떻게 해야 할까요?

```cpp
// Launch kernel with multiple block sizes
SomeKernel<<<N/128, 128>>>(parameters..);
SomeKernel<<<N/64, 64>>>(parameters..);
SomeKernel<<<N/32, 32>>>(parameters..);
```

이때 `extern` 키워드를 사용하면 됩니다. 커널 런칭을 할 때, 공유 메모리 사이즈를 명시해주어야 합니다. 

```cpp
__global__ void someKernel(int* A)
{
    extern __shared__ int sm[];
}
```












## Reference
- https://mkblog.co.kr/nvidia-gpu-memory-coalescing-coalesced-memory-access/