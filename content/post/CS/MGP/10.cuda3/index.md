---
title: "CUDA 메모리 계층 구조"
date: 2024-05-29
draft: false
summary : "GPU 아키텍쳐 살펴보기"
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

본 포스팅에서는 GPU가 CUDA 프로그램을 어떻게 처리하는지에 대해 살펴보겠습니다. 세부적으로는 GPU 하드웨어 구조, CUDA 스레드 모델과의 관계에 대해 다룹니다.

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

    그리드에 포함된 스레드 블록은 그리드가 배정된 GPU 속 SM에 의해 처리됩니다. 즉, 스레드 블록을 처리하는 단위는 SM입니다. 이때 블럭 내에 있는 스레드들은 SM에서 같이 실행됩니다. 그렇다면 각 SM에 스레드 블럭을 어떻게 할당할까요? SM이 2개인 경우, SM마다 4개의 스레드 블럭이 할당됩니다. SM이 4개인 경우에는 SM마다 2개의 스레드 블럭이 할당됩니다. 이처럼 하나의 SM에 여러 개의 블럭이 할당될 수 있는데, SM이 갖는 자원의 양과 한 스레드 블럭을 처리하기 위해 필요한 자원의 양에 따라 한 SM이 동시에 처리할 수 있는 스레드 블록 수가 결정됩니다. SM에 할당된 블록 중 현재 필요한 자원을 모두 할당받고 실행할 수 있는 상태인 스레드 블록을 **활성 블록(active block)**이라고 합니다. 

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

최대 블록 개수는 32개이므로, 총 스레드 개수는 32 * 32 = 1024개 입니다. 이때 최대 스레드 대비 효율은 50% 입니다.

2. 768 threads / block

2개의 블록이면 1536개 스레드이고, 3개의 블록이면 2304개의 스레드로 최대 허용 가능 스레드 개수인 2048개를 능가합니다. 따라서 가능한 총 스레드 개수는 1536개로, 효율은 75%입니다.

3. 256 threads / block

8개의 스레드 블록을 사용하면 2048개의 스레드를 사용할 수 있습니다. 이때 효율은 100% 입니다. 


이번 포스팅까지 CUDA 의 기초적인 내용과 GPU의 하드웨어 대해 다루어봤습니다. 다음 포스팅부터는 matmul 연산을 통해 CUDA 프로그램을 최적화 하는 방법에 대해 다루어 보겠습니다.


## Reference

- https://www.nvidia.com/content/pdf/fermi_white_papers/p.glaskowsky_nvidia's_fermi-the_first_complete_gpu_architecture.pdf
- CUDA 기반 GPU 병렬 처리 프로그래밍, ch.6 김덕수 저
- Multicore and GPU Programming, 연세대학교 박영준 교수님