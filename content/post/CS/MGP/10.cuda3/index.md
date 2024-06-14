---
title: "CUDA와 Nvidia GPU 아키텍처: 스레드 계층, 메모리 계층 및 GPU 캐시 구조 이해하기"
date: 2024-05-29
draft: false
summary : "CUDA 의 스레드 계층와 GPU 하드웨어의 관계, 메모리 계층, GPU 캐시 구조"
tags: ["Multicore-GPU-Programming", "cuda"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 10
slug : "cuda-programming-3"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

본 포스팅에서는 GPU가 CUDA 프로그램을 어떻게 처리하는지에 대해 살펴보겠습니다. 세부적으로는 GPU 캐시 구조, CUDA 스레드 모델과의 관계, CUDA 메모리 계층에 대해 다룹니다. 

## Nvidia GPU Architecture

CUDA를 지원하는 엔비디아의 GPU 아키텍쳐는 2008년부터 나와서, 2010년에는 페르미(Fermi), 2020년에는 암페어(Ampere) 아키텍쳐가 등장했습니다. 페르미는 조금 오래되었지만 CUDA 관점에서 구조가 정립된 첫번째 아키텍쳐입니다. 따라서 지금까지 큰 틀은 변함 없습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b7b18010-e03f-4542-a619-5656ee4f91b1 'Fermi Architecture, Source : Nvidia')


### Streaming Multiprocessor

SM은 Streaming Multiprocessor의 약자로 하나의 GPU는 여러 개의 SM을 포함합니다. SM은 여러 개의 CUDA 코어를 가진 연산 장치입니다. 페르미 아키텍쳐의 경우 하나의 SM에 32개의 CUDA 코어를 가지고 있습니다. 


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a7b888ad-b5d5-4fd4-bc0d-f81167458f5e 'Nvidia Pascal GP104 Streaming multi-processor unit')

### CUDA core

CUDA 코어는 GPU의 가장 기본적인 프로세싱 유닛으로, 연산을 수행하는 가장 기본 단위입니다. 위의 페르미 구조에서 코어를 확대한 것을 보면, 안에 FP unit, INT unit 과 같이 실제 연산 유닛을 확인할 수 있습니다. CUDA 코어 하나가 스레드 하나를 처리합니다.  

## CUDA 스레드 계층과 GPU 하드웨어 간 관계

- **그리드 -> GPU**

    CUDA 커널이 실행되면 스레드 레이아웃에 따라 그리드가 생성됩니다. 이 그리드가 GPU를 사용하는 단위입니다. 

- **스레드 블록 -> SM**

    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/54c83ce4-78c6-4fb9-b76d-bc145bb1795b 'Blocks and SM')

    그리드에 포함된 스레드 블록은 그리드가 배정된 GPU 속 SM에 의해 처리됩니다. 즉, 스레드 블록을 처리하는 단위는 SM입니다. 이때 블럭 내에 있는 스레드들은 SM에서 같이 실행됩니다. 그렇다면 각 SM에 스레드 블럭을 어떻게 할당할까요? SM이 2개인 경우, SM마다 4개의 스레드 블럭이 할당됩니다. SM이 4개인 경우에는 SM마다 2개의 스레드 블럭이 할당됩니다. 이처럼 하나의 SM에 여러 개의 블럭이 할당될 수 있는데, SM이 갖는 자원의 양과 한 스레드 블럭을 처리하기 위해 필요한 자원의 양에 따라 한 SM이 동시에 처리할 수 있는 스레드 블록 수가 결정됩니다. SM에 할당된 블록 중 현재 필요한 자원을 모두 할당받고 실행할 수 있는 상태인 스레드 블록을 **활성 블록(active block)** 이라고 합니다. 

    스레드 블록 내에 있는 스레드들은 논리적으로 "같이" 실행됩니다. 그리고 서로 간에 소통할 수 있는데, 이것은 `__syncthread` 명령어와 공유 메모리를 사용해서 할 수 있습니다. 

    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8e6ac00b-2870-437a-91e7-2ba4d9872658 'Communication between threads')

    
    그런데 블럭 사이에는 디펜던시가 없어서 독립적으로 실행될 수 있고, 실행되는 순서가 없습니다. 그리고 더 많은 SM이 있을 수록 성능이 더 좋아집니다. 

- **워프 & 스레드 -> SM 속 CUDA 코어**

    스레드 블록에 포함된 스레드들은 32개의 스레드로 구성된 **워프**로 분할됩니다. 페르미 아키택쳐의 경우 32개의 CUDA 코어가 있으며, 16개씩 2개의 그룹으로 분리되어 있습니다. 각 CUDA 코어 그룹이 하나의 워프를 처리합니다. SM 내부의 코어 수는 아키텍쳐에 따라 다른데, 대체로 32의 배수이며 이것은 워프가 32개의 스레드로 구성되었기 때문입니다. 만약 블록 사이즈가 32의 배수가 아니면, 몇개의 CUDA 코어는 idle 상태가 될 것입니다. 

    <img width="795" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f085b30e-5483-43ce-8c4f-e5cdd3552e39">

    워프는 하나의 명령어에 의해 움직입니다. 워프 스케쥴러(Warp Scheduler)와 명령어 전달 유닛(Dispatch Unit)들이 다음에 처리할 워프를 결정하거나 명령을 내리고, 각 CUDA 코어는 명령어에 따라 스레드의 작업을 처리합니다. 

    워프 내 스레드들은 하나의 명령어에 의해 움직이지만, **각 스레드는 독립적으로 처리될 수도 있습니다**. 각 스레드는 자신만의 실행 문맥(execution context)를 갖습니다. SIMD 구조에서는 하나의 실행 문맥 또는 하나의 스레드에서 하나의 명령어로 여러 개의 데이터를 처리합니다. 반면, GPU에서는 자신만의 실행 문맥을 가진 스레드가 하나의 명령어에 의해 제어된다는 점에서 SIMD가 아닌 **SIMT**로 분류됩니다.

    
### Zero Context switching overhead 

**SM은 워프를 스케쥴링하는데 비용이 들지 않습니다.**  CPU가 컨텍스트 스위칭을 할 때는, 각 코어에 레지스터 집합에 프로세스의 작업 상황을 저장하고 새로운 프로세스의 상황을 레지스터에 복원해야 했습니다. 그러나 SM은 워프에 필요한 모든 정보(PC + regs file)를 SM 안에 담고 있기 때문에 컨텍스트 스위칭 비용이 0에 가깝습니다. 

SM 안에는 여러 개의 워프가 있고 워프는 서로 번갈아 CUDA 코어를 사용합니다. 워프가 전환되는 과정에서 CUDA 코어가 참조하는 문맥도 바뀌어야 합니다. 하지만 다른 연산 장치와 달리 문맥을 메모리에 복사하거나 메모리에서 읽어오는 작업이 필요하지 않습니다. 그 이유는 블록 내 스레드는 SM의 레지스터 파일을 나누어서 사용하기 때문입니다. 즉 각 스레드는 본인만의 전용 책상을 가지고 있다고 이해하면 됩니다. 따라서 스레드가 잠시 일을 멈추더라도 스레드의 문맥(레지스터 값)은 유지되며, 다시 CUDA 코어를 할당받으면 다른 작업 없이 바로 연산을 다시 시작할 수 있습니다. 

따라서 일반적으로 CPU 연산에서는 코어 수와 비슷한 수의 스레드를 만들어서 실행하는 것이 권장되지만, GPU 연산시에는 코어 수의 3~4배에서 많게는 10배까지 스레드를 사용하기를 권장합니다.

![Image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/31554ff8-6f07-4694-a380-d548dd861fe6 'SM 내부 스레드 사이 문맥 교환')



### Warp Divergence

스레드 블록에서, 스레드는 워프 단위로 실행됩니다. 그리고 이것은 워프가 실행할 준비가 되면 스케쥴링 됩니다. 따라서 워프의 실행 순서는 보장되지 않습니다. 워프 내에 모든 스레드는 같은 명령어를 실행하는데, 이때 아래 코드처럼 **분기(branch)** 명령어가 있으면 어떻게 할까요? 

```cpp
__global__
void kernel_with_branch(int *_output)
{
    if (threadIdx.x % 2 == 0)
        _output[threadIdx.x] = 1;
    else
        _output[threadIdx.x] = 2;
}
```

여기서는 `threadIdx.x` 가 홀수인 스레드와 짝수인 스레드가 서로 다른 명령어를 수행해야 합니다. 하지만 GPU는 워프에게 한 번에 하나의 명령어만 지시할 수 있습니다. 이런 경우 GPU는 **각 분기를 순차적으로 처리**하는 방식으로 분기를 실행합니다. 하지만 이처럼 분기에서 2가지 경로가 있으면 하나의 워프를 두 번에 나누어 처리하게 됩니다. 따라서 분기가 없는 경우 대비 연산이 2배 정도 느려집니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/20ee9281-5499-441e-8c57-d5d2987e62e5 'warp execution in branch instruction')

최악의 경우 32개의 스레드가 모두 다른 분기를 따르게 되면, 최대 32배까지 연산 속도가 느려질 수 도 있습니다. 따라서 워프 분기는 CUDA 프로그램 성능을 크게 떨어뜨리며, CUDA 알고리즘을 설계할 때는 분기를 최대한 피하는 것이 좋습니다. 


## Latency Hiding

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/83b8fb19-19d7-4981-afc5-cc486b14b0fa '일반적인 데이터 처리 과정')

일반적으로 데이터 처리는 위와 같이 메모리 접근과 연산을 반복하는 구조입니다. 처리한 데이터를 읽어오거나 결과를 쓰기 위해 메모리에 접근하는 시간 동안 CUDA 코어와 같은 연산 장치는 아무 일도 하지 못하고 쉬게 됩니다. 이처럼 메모리 접근 작업이 끝날 때까지 연산 코어가 대기하는 시간을 메모리 **접근 대기 시간(memory access latency)** 라고 합니다. 

GPU는 수백 ~ 수만개의 코어를 갖고 있기 때문에 이러한 **접근 대기 시간을 최소화**하는 것이 중요합니다. 이를 위해서 GPU는 하드웨어 전략과 소프트웨어 전략을 사용하고 있습니다. 

1. **하드웨어 전략 : 높은 대역폭의 메모리**

||CPU||GPU||
|--|---|---|---|---|
||Intel i9 11900K|AMD Lizen 9 5950X|Nvidia RTX 3080|Radeon RX 6800|
|연산 코어 수|8|16|8704|3840|
|메모리 타입|DDR4-3200|DDR4-3200|GDDR6X|GDDR6|
|메모리 크기|최대 128GB|최대 128GB| 10~12GB| 16GB|

GPU는 CPU 대비 높은 대역폭의 메모리를 사용해서 필요한 데이터를 빠르게 공급 받을 수 있습니다. 

2. **소프트웨어 전략 : CUDA 코어 수 보다 많은 스레드 사용하기** 

<img width="538" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/467bc3e4-779b-47d7-9bb9-a4452b9ab096">

CUDA 코어 수 대비 많은 스레드를 사용해서 한 스레드가 메모리 대기하는 동안 다른 스레드가 CUDA 코어를 사용하게끔 합니다. 이것이 가능한 이유는 위에서 설명했듯, 문맥 교환 비용이 거의 0에 가깝기 때문입니다. 

예시를 하나 봅시다. 아래의 하드웨어 스펙에서, **블록 사이즈를 어떻게 설정**해야 할까요?

```
- Max num of threads : 2048
- Max warps : 64
- Max thread blocks : 32
- Registers : 65536
- Shared memory : 96KB
```

1. 32 threads / block

최대 블록 개수는 32개(2048 / 64 = 32)이므로, 총 스레드 개수는 32 * 32 = 1024개 입니다. 이때 최대 스레드 대비 효율은 50% 입니다.

2. 768 threads / block

2개의 블록이면 1536개 스레드이고, 3개의 블록이면 2304개의 스레드로 최대 허용 가능 스레드 개수인 2048개를 능가합니다. 따라서 가능한 총 스레드 개수는 1536개로, 효율은 75%입니다.

3. 256 threads / block

8개의 스레드 블록을 사용하면 2048개의 스레드를 사용할 수 있습니다. 이때 효율은 100% 입니다.


## CUDA 메모리 계층

CUDA 에서는 메모리를 접근 가능 범위에 따라 스레드 수준, 블록 수준, 그리드 수준 메모리로 구분합니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6803618f-67b9-4ea2-a87c-a80e49cf43d5 'CUDA Memory Hierarchy')


### 스레드 수준 메모리(per-thread memory)

스레드 수준 메모리는 **각 스레드 내부에서 사용**되므로, 다른 스레드에서 접근할 수 없는 공간입니다. 스레드 수준 메모리에는 레지스터와 지역 메모리가 있습니다. 

- **레지스터** 

    레지스터는 CUDA 코어 연산을 위한 데이터를 담아두고 사용하는 공간입니다. 레지스터는 커널 내부에서 선언된 지역변수를 위해 사용됩니다. 레지스터는 접근 속도가 가장 빠르지만(~1 GPU cycle) 가장 크기가 작습니다. 일반적으로 블록 또는 SM 하나 당 8K~64K개의 4Byte 레지스터를 가집니다. 


### 블록 수준 메모리(per-block memory)

블록 수준 메모리는 **블록 내 모든 스레드들이 접근할 수 있는 공유 메모리**입니다. 블록 수준 메모리는 SM 내부에 있는 온-칩 메모리입니다. 접근 속도는 1~4 GPU cycles 로 빠릅니다. 크기는 디바이스 메모리보다 작으며 compute compatibility 에 따라 SM 당 16 ~ 96KB 의 크기를 가집니다. 크기는 작지만 접근 속도가 매우 빠른 공유 메모리 공간을 어떻게 활용하는지에 따라 행렬곱의 성능이 크게 빨라질 수 있습니다. 

공유 메모리는 블록 내 모든 스레드가 공유할 수 있어서, 블록 내 스레드들 사이 데이터 공유 통로의 역할도 합니다. 하지만 서로 다른 블록끼리는 다른 공유 메모리 영역을 사용하기 때문에, 다른 블록에 속한 스레드끼리는 서로의 데이터를 볼 수 없습니다. 

공유 메모리는 사용자가 직접 관리하는 사용자 관리 캐시 형태로 사용할 수 있습니다. 이때 `__shared__` 명령어를 사용해 명시적으로 선언하고 할당받아 사용할 수 있습니다. 이때 **정적 할당**과 **동적 할당** 2가지 방법으로 할당받을 수 있습니다.

- **정적 할당**

    ```cpp
    __global__
    void kernel(void)
    {
        __shared__ int sharedMemory[512];
    }
    ```

    `__shared__` 가 붙은 변수 선언의 경우, 각 스레드의 지역 변수가 아닌 스레드 블록 내 모든 스레드가 공유하는 변수로 선언됩니다. 정적 할당의 경우 CUDA 프로그램이 컴파일 될 때 그 크기가 결정됩니다. 프로그램의 상황 또는 compute capability 와 같은 GPU 사양에 따라 사용할 공유 메모리의 크기를 변경하고 싶다면 동적 할당 방법을 사용할 수 있습니다.

- **동적 할당**

    ```cpp
    extern __shared__ int sharedMemory[];
    __global__ 
    void kernel(void)
    {...}

    int main(void)
    {
        int size = 512;
        kernel <<<gridDim, blockDim, sizeof(int) * size>>>();
    }
    ```

    공유 메모리의 크기를 동적으로 조절하는 경우, 크기가 정해지지 않는 extern 배열 형태로 커널 밖에서 선언해야 합니다. 이때 sharedMemory 배열의 크기는 실행 구성의 3번째 인자(`sizeof(int) * size`)에 의해 결정되며, 커널 실행 시 메모리 공간이 할당됩니다. 

    ```cpp
    extern __shared__ int sharedPool[];
    int *sIntArray = sharedPool;
    float *sFloatArray = (float*)&sharedPool[sizeIntArr];

    __global__
    void kernel(void)
    {
        ...
        sIntArray[threadIdx.x] = 0;
        sFloatArray[threadIdx.x] = 0.0f;
    }

    int main(void)
    {
        int size = 512;
        kernel <<<gridDim, blockDim, sizeof(int) * sizeIntArr + sizeof(float) * sizeFloatArr>>>();
    }
    ```

    여러 개의 공유 메모리 공간을 할당해야 하는 경우에는 모든 배열의 크기만큼의 공간을 가지는 하나의 큰 공유 메모리 배열을 선언하고, 포인터를 이용해 해당 공간을 분할하는 방법을 사용해야 합니다. 


### 그리드 수준 메모리(per-grid memory)

그리드 수준 메모리는 **그리드 내 모든 스레드가 접근할 수 있는 메모리 영역**입니다. 그리드 수준 메모리에는 전역 메모리와 상수 메모리, 텍스쳐 메모리가 있습니다. 세 종류 메모리 모두 디바이스 메모리 공간을 사용합니다. 전역 메모리는 읽기/쓰기가 모두 가능하지만, 상수 메모리와 텍스쳐 메모리는 특수 목적 메모리로, 온-칩 캐시를 활용하며 읽기만 가능합니다. 

- **전역 메모리**

    전역 메모리(global memory)는 **모든 스레드가 접근 가능한 메모리 영역**입니다. GPU 메모리 중 가장 큰 공간을 차지하지만 접근 속도는 500 GPU cycle 수준으로 가장 느립니다. 

    전역 메모리는 호스트에서 접근 가능한 GPU 메모리입니다. 호스트에서 `cudaMalloc()`을 통해 요청한 메모리 공간이 할당되는 곳이 전역 메모리 공간이며, `cudaMemcpy()` 를 통해 호스트-디바이스 사이 데이터를 복사한다는 것은 전역 메모리에 있는 데이터를 복사한다는 의미입니다. 


- **상수 메모리**

    ```cpp
    __constant__ int constMemory[512]; // global-socpe 로 선언

    __global__ 
    void kernel(void)
    {
        ...
        int a = constMemory[threadIdx.x]; // Okay (Read)
        constMemory[threadIdx.x] = 0; // Error! (Write)
    }

    int main(void)
    {
        int table[512] = {0};
        cudaMemcpyToSymbol(constMemory, table, sizeof(int) * 512); // 상수 메모리 초기화
        ...
        kernel <<<gridDim, blockDim>>>();
    }
    ```

    상수 메모리(constant memory) 는 **변하지 않는 값을 저장하는 공간**입니다. 따라서 쓰기 연산은 불가능한 읽기 전용 메모리입니다. 상수 메모리는 디바이스 메모리 공간에 저장되고, 최대 크기는 64KB입니다. 그러나 전역 메모리와 달리 온-칩 메모리인 상수 캐시(constant cache)를 사용합니다. 상수 캐시의 크기는 compute capability에 따라 다르며, 대략 48KB 수준입니다. 상수 메모리에 대한 접근은 캐싱되며, 캐시 히트가 높을 경우 접근 속도가 굉장히 빨라집니다(~ 5 GPU cycles).


- **텍스쳐 메모리**

    텍스쳐 메모리(texture memory)는 GPU 원래 기능인 그래픽스 연산을 위해 사용되는 메모리입니다. 디바이스 메모리 공간을 사용하며, 전용 텍스쳐 캐시를 가집니다. 이 영역을 CUDA 프로그램에서 사용하는 경우는 드뭅니다. 

## GPU 캐시 


**GPU 캐시는 아주 작지만 매우 빠른 메모리 공간**으로, 자주 쓰는 데이터를 캐싱해놔서 데이터 접근 시간(data access latency)를 줄이는 역할을 합니다. 하지만 어떤 것이 캐싱될 것인지는 개발자가 결정할 수 없고, CPU나 GPU 하드웨어에 의해 관리되어서 **하드웨어 관리 메모리(hardware managed memory)** 라고도 합니다.


![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0d517b3d-6496-4cbe-a1bb-fa55a61cd254 'GPU cache structure')

L2 캐시는 모든 SM 들이 공유하는 캐시 공간이고, L1 캐시는 SM 마다 할당된 캐시 공간입니다. GPU의 L1,L2 캐시는 CPU의 캐시와 유사하지만, CPU에서는 한 번에 하나의 스레드가 데이터 접근을 요청하지만 GPU에서는 워프(32개의 스레드)가 동시에 메모리 접근을 요청합니다. 

L1 캐시는 SM 내부의 온-칩 메모리인 공유 메모리의 공간을 사용합니다. 따라서 각 커널이 SM 내부 온-팁 메모리를 L1캐시와 스레드 블록을 위한 공유 메모리에 어느 정도 사용할지 조절할 수 있습니다. L1 캐시 크기는 `cudaFuncSetCacheConfig()` 함수를 통해 설정할 수 있습니다. 

```cpp
cudaFuncSetCacheConfig(커널 이름, 분배 정책)
```


```cpp
// Device code
__global__
void myKernel()
{
    ...
}

// Host code
// Runtime API
// cudaFuncCachePreferShared : shared memory is 48KB
// cudaFuncCachePreferEqual : shared memory is 32KB
// cudaFuncCachePreferL1 : shared memory is 16KB
// cudaFuncCachePreferNone : no preference
cudaFuncSetCacheConfig(MyKernel,cudaFuncCachePreferShared);
```

지금까지 CUDA 메모리 계층을 살펴봤는데, 요약하자면 아래와 같습니다.

|공유 범위|메모리 종류|지원 연산|접근 속도|캐시 지원|크기|
|-------|-------|-------|-------|---------|--|
|Thread|Register|Read,Write|Fastest|X|Smallest|
||Local Memory|Read,Write|Slow|∆|*|
|Block|Shared Memory|Read,Write|Fast|X|Small|
|Grid(global)|Global Memory|Read,Write|Slow|∆|Largest|
||Constant Memory|Read-only|Fast|O(전용 캐시)|Small|
||Texture Memory|Read-only|Fast|O(전용 캐시)|Small|

∆ : Compute Capability 및 캐시 설정에 따라 달라질 수 있음

\* : 디바이스 메모리 영역을 사용함

<img width="785" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2daefdf1-6b11-4c77-bd7c-597cccc8d8f1">


---

이번 포스팅까지 CUDA 의 스레드 계층, 메모리 계층, GPU 캐시 구조 대해 다루어봤습니다. 또, CUDA 의 성능 고려 사항 중 SM 당 스레드 개수를 설정하는 것과, 브랜치 분기를 피하는 이유에 대해 살펴보았습니다. 다음 포스팅에서는 **CUDA의 메모리(공유 메모리, 레지스터, 글로벌 메모리)와 매트릭스 연산을 최적화하는 방법들**에 대해 더 다루어 보겠습니다.


## Reference

- https://www.nvidia.com/content/pdf/fermi_white_papers/p.glaskowsky_nvidia's_fermi-the_first_complete_gpu_architecture.pdf
- CUDA 기반 GPU 병렬 처리 프로그래밍, ch.6,8 김덕수 저
- Multicore and GPU Programming, 연세대학교 박영준 교수님