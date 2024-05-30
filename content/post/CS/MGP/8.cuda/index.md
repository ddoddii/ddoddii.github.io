---
title: "CUDA 프로그래밍 기초와 CUDA 메모리 계층구조"
date: 2024-05-04
draft: false
summary : "Hello, CUDA!"
tags: ["Multicore-GPU-Programming", "cuda"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 8
slug : "cuda-programming"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

본 포스팅에서는 **GPU와 CPU의 차이, CUDA 프로그래밍의 기초**에 대해 알아보겠습니다.

## GPU를 사용하는 이점

그래픽 처리 장치(GPU)는 비슷한 가격과 전력 소모 내에서 CPU보다 훨씬 높은 명령 처리량과 메모리 대역폭을 제공합니다. 많은 애플리케이션은 이러한 더 높은 성능을 활용하여 CPU보다 GPU에서 더 빠르게 실행됩니다. FPGA와 같은 다른 컴퓨팅 장치들도 에너지 효율이 높지만, GPU에 비해 프로그래밍 유연성이 훨씬 낮습니다.

GPU와 CPU 간의 이러한 성능 차이는 서로 다른 목표를 염두에 두고 설계되었기 때문입니다. CPU는 가능한 한 빠르게 일련의 작업(스레드)을 실행하는 데 뛰어나도록 설계되었으며, 수십 개의 이러한 스레드를 병렬로 실행할 수 있습니다. 반면, GPU는 수천 개의 스레드를 병렬로 실행하는 데 뛰어나도록 설계되었습니다(단일 스레드 성능이 느리더라도 이를 분산 처리하여 더 높은 처리량을 달성합니다).

GPU는 고도로 병렬적인 계산을 위해 특화되어 있으며, 데이터 캐싱과 흐름 제어보다는 데이터 처리에 더 많은 트랜지스터가 할당되도록 설계되었습니다. 아래의 그림은 CPU와 GPU의 칩 자원 분포 예시를 보여줍니다.

<img width="640" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4d68cd62-7359-4975-bd5c-1e85045ad27a">

데이터 처리, 예를 들어 부동 소수점 계산에 더 많은 트랜지스터를 할당하는 것은 고도로 병렬적인 계산에 유익합니다. GPU는 대규모 데이터 캐시와 복잡한 흐름 제어에 의존하지 않고 계산으로 메모리 접근 지연을 숨길 수 있으며, 이는 트랜지스터 측면에서 비용을 아낄 수 있습니다.

일반적으로 애플리케이션은 병렬 부분과 순차 부분이 혼합되어 있기 때문에 시스템은 전체 성능을 최대화하기 위해 GPU와 CPU를 혼합하여 설계됩니다. 병렬성이 높은 애플리케이션은 GPU의 대규모 병렬 특성을 활용하여 CPU보다 더 높은 성능을 달성할 수 있습니다.


## CUDA는 무엇인가?

CUDA는 Compute Unified Device Architecture로, 엔비디아에서 GPU를 GPGPU 목적으로 사용할 수 있게 제공하는 프로그래밍 인터페이스 입니다. CUDA를 통해 GPU를 직접 제어하려면 CUDA C/C++로 코드를 작성해야 합니다.

CUDA는 엔비디아에서 개발했기 때문에 기본적으로 엔비디아의 GPU에서만 사용 가능하며, 다른 회사의 제품에서는 사용할 수 없습니다. 다른 회사의 GPU를 GPGPU 용도로 사용하려면 OpenCL 등과 같은 개방형 API를 사용해야 합니다. 

### 내 GPU 확인하기

우선 CUDA를 사용하려면 내 컴퓨터 혹은 서버에 있는 GPU 사양을 확인해야 합니다. 리눅스 서버 상에서는 간단하게 `nvidia-smi` 명령어로 확인할 수 있습니다. 

<img width="577" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/f538c4e1-517c-4f37-8f84-18120a803591">

실제 제품이름을 알고 싶으면 `nvidia-smi -q | grep -i 'Product Name'` 을 입력하면 됩니다.


<img width="500" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/e6cd9319-0de1-46fc-acbf-3c6a8969b926">

그 후 Nvidia 의 [GeForce RTX 3090 spec](https://www.nvidia.com/en-us/geforce/graphics-cards/30-series/rtx-3090-3090ti/) 을 보면 GPU Engine과 메모리의 스펙을 볼 수 있습니다. 

<img width="901" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/0f3e6f07-58a4-4f26-87dc-12f81aa73fe6">

### CUDA 개발 환경 설정하기 

CUDA 프로그램을 작성하고 컴파일하기 위해서는 CUDA 툴킷이 필요합니다. 이것은 [CUDA 툴킷 페이지](https://developer.nvidia.com/cuda-downloads)에서 본인의 운영체제, 시스템 아키텍쳐, CUDA 버전에 맞게 다운 받을 수 있습니다. 설정을 마치고 `nvcc --version` 을 입력해서 정상적으로 설치되었는지 확인할 수 있습니다.

<img width="339" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/18594236-e729-4e21-bda7-5c9721f446cd">

## 첫번째 프로그램 작성해보기

이어서 CUDA를 사용해 첫번째 프로그램을 작성해보겠습니다. 

CUDA 프로그래밍을 하면, 호스트(host)와 디바이스(device)의 개념이 자주 나옵니다. 일반적으로 호스트는 CPU를, 디바이스는 GPU를 뜻합니다. 왜냐면 GPU에서 실행되는 최초 모듈은 CPU에 의해 호출되기 때문입니다. 따라서 호스트 코드는 CPU에서 실행되는 코드를, 디바이스 코드는 GPU에서 실행되는 코드를 의미합니다. 

가장 기본적인 CUDA 프로그램은 아래와 같이 생겼습니다.

```cpp
#include "cuda_runtime.h"
#include "device_launch_parameters.h"
#include <stdio.h>

__global__ void helloCUDA(void)
{
    printf("Hello CUDA from GPU!\n");
}

int main(void)
{
    printf("Hello GPU from CPU!\n");
    helloCUDA<<<1, 10>>>();

    // Synchronize the device
    cudaError_t err = cudaDeviceSynchronize();

    // Check for errors
    if (err != cudaSuccess)
    {
        printf("CUDA error: %s\n", cudaGetErrorString(err));
    }

    return 0;
}
```

이 코드를 보면서 CUDA 문법을 알아가 봅시다.

### CUDA C/C++ 키워드

CUDA는 C/C++를 확장한 프로그래밍 인터페이스입니다. 위의 코드에서 등장한 `__global__`과 `<<<>>>` 는 확장 키워드의 대표적인 예시입니다. `__global__` 는 호스트에서 호출하고, 디바이스에서 실행되는 함수임을 지칭하는 키워드입니다. 함수의 호출자와 실행 공간을 지정하는 키워드는 아래와 같습니다.

|키워드|함수의 호출자|실행 공간|
|----|---------|--------|
|`__host__`| host | host|
|`__device__`|device|device|
|`__global__`|host|device|

`__global__` 키워드로 지정된 함수는 호스트에서 호출하고 디바이스에서 실행되는데, CUDA에서는 이것을 **커널**이라고 합니다. 커널은 호스트가 디바이스에게 연산을 요청하는 통로이며, CUDA 스레드들의 동작을 정의하는 함수입니다. 커널 호출 시에는 커널을 수행할 CUDA 스레드의 수를 지정해주어야 합니다. CUDA 스레드 수는 `<<<...>>>` 실행 구성 문법을 통해 지정합니다. 위의 코드는 `helloCUDA<<<1, 10>>>();` 에서 10개의 CUDA 스레드가 `helloCUDA()` 커널을 수행하라는 의미입니다. 

위의 코드를 컴파일하고 실행해보면 아래와 같은 결과가 나오는 것을 볼 수 있습니다.

<img width="244" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/15d4b601-374a-49d6-b323-48915c4414ee">

## CUDA 프로그램의 구조와 흐름

CUDA 프로그램은 호스트 코드와 디바이스 코드로 구성되어 있습니다. GPU 등 다른 연산 장치를 활용하기 위해서는 호스트 코드에서 커널을 호출해야 합니다. 컴퓨터 시스템의 기본 메모리 공간 역시 CPU가 사용하는 시스템 메모리(일반적으로 DRAM)입니다. 


1. **호스트 메모리에서 디바이스 메모리로 입력 데이터 복사**

    <img width="701" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/16f3a37e-dca3-459c-ac53-a0a0eb1f7a18">


    CPU와 GPU는 서로 독립된 장치로, 사용하는 메모리 영역도 다릅니다. 따라서 GPU를 사용하기 위해서는 호스트 메모리에 있는 데이터를 디바이스 메모리로 복사해주어야 합니다. 

2. **GPU 연산** 

    <img width="725" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/9859ab5a-99c7-4486-8b43-9545b94261f7">


    연산에 필요한 메모리가 디바이스 메모리에 준비된 후, GPU 연산이 이루어 집니다. GPU 연산은 커널 호출을 통해 시작되며 디바이스 메모리에서 관리합니다. 

3. **디바이스 메모리에서 호스트 메모리로 연산 결과 복사**

    <img width="672" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/0d6e502b-031b-44f7-94da-d22a2e5ec0b3">

    GPU 연산이 끝나면 연산 결과를 다시 호스트 메모리로 복사합니다.

### CUDA 기초 메모리 API

기본 흐름을 알았으니 메모리 복사와 관련된 API를 살펴봅시다. 

#### 디바이스 메모리 공간 할당 및 초기화 API

- 디바이스 메모리 할당 : `cudaMalloc()`

    ```cpp
    cudaError_t cudaMalloc(void **ptr, size_t size)
    ```

    - `ptr` : 디바이스 메모리 공간의 시작 주소를 담을 포인터 변수의 주소
    - `size` : 할당할 공간의 크기(byte)

- 디바이스 메모리 해제 : `cudaFree()`

    ```cpp
    cudaError_t cudaFree(void *ptr)
    ```

    - `ptr` : 해제할 메모리 공간을 가리키는 표인터 변수 

- 디바이스 메모리 초기화 : `cudaMemset()`

    ```cpp
    cudaError_t cudaMemset(void *ptr, int value, size_t size)
    ```

    - `ptr` : 초기화할 메모리 공간의 시작 주소
    - `value` : 해당 공간의 각 바이트를 초기화할 값
    

- 에러 코드 확인 : `cudaGetErrorName()`

    - CUDA API 의 반환값은 대부분 `cudaError_t` 의 에러코드입니다. 따라서 이 함수를 이용해서 에러코드를 확인할 수 있습니다.


#### 디바이스 메모리 공간 할당 및 초기화 API

- 장치 간 데이터 복사 : `cudaMemcpy()`

    ```cpp
    cudaError_t cudaMemcpy(void *dst, const void *src, size_t size, enum cudaMemcpyKind kind)
    ```
    
    - `dst` : 데이터가 복사될 메모리 공간(destination)의 시작 주소를 담고 있는 포인터 변수
    - `src` : 복사할 원본 데이터가 들어있는 메모리 공간의 시작 주소를 담고 있는 포인터 변수 
    - `size` : 복사할 데이터의 크기 
    - `kind` : 데이터 복사 방향을 설정하는 인자 


        |cudaMemcpyKind|복사 방향|
        |----|----|
        |cudaMemcpyHostToHost| 호스트 메모리 -> 호스트 메모리|
        |cudaMemcpyHostToDevice| 호스트 메모리 -> 디바이스 메모리|
        |cudaMemcpyDeviceToHost| 디바이스 메모리 -> 호스트 메모리|
        |cudaMemcpyDeviceToDevice| 디바이스 메모리 -> 디바이스 메모리|
        |cudaMemcpyDefault| dst 와 src 의 포인터 값에 의해 결정  (unified virtual addressing을 지원하는 경우에만 사용가능)|


본 포스팅에서는 CUDA 프로그래밍의 기초에 대해 알아보았고, 다음 포스팅에서는 실제로 CUDA 병렬 프로그래밍 작성시 생각해볼 점들에 대해 다루어보겠습니다. 

작성한 실제 코드는 아래 레포지토리의 **cuda 폴더**에서 볼 수 있습니다.

{{< github repo="ddoddii/Multicore-GPU-Programming" >}}





## Reference
- https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html
- CUDA 기반 GPU 병렬 처리 프로그래밍, 김덕수 저
