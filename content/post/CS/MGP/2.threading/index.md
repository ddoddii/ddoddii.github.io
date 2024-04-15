---
title: "c++로 알아본 쓰레드 관리하기"
date: 2024-03-20
draft: false
summary : "c++에서 쓰레드 관리하기, 쓰레드 안전 구현하는 방법"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 2
slug : "threading"
toc : true
katex : true
markup: 'mmark'
---

## Programming Model

병렬 프로그래밍에서, 프로그래밍 모델은 사용자에게 communication abstractions 를 제공해줍니다. 여기에는 **shared memory model** 과 **message-passing model** 이 있습니다. 

### Shared Memory Model

#### Memory

우선 **메모리**는 무엇일까요? 메모리는 주소에 의해 접근(읽기/쓰기)될 수 있는 바이트들의 집합입니다. 아래의 메모리에서 0x8 주소에는 32 가 저장되어 있고, 0x10 에는 128이 저장되어 있습니다. 이때 byte-addressable 이라고 가정합니다. 모든 주소는 1byte(8bit)를 저장합니다. 각 셀은 8byte 를 가지고 있습니다. 하지만 메모리를 읽을 때 1bit 단위로 읽을 수 없고, 8bit 단위로 읽어야 합니다. 

파일 시스템의 유닛은 **page** 라고 합니다. 파일시스템은 page addressable memory 를 사용합니다. 

<img width="208" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f76ee624-7f9f-4311-8c12-f20f6f8490a2">

우선 하드웨어는 신경쓰지 않고,  모든 쓰레드들이 같은 주소공간을 공유(shared memory)한다고 가정합시다. 쓰레드들이 같은 주소에 접근하면, 모두 같은 정보를 볼 수 있습니다. 하나의 쓰레드가 주소 공간에 어떠한 값을 쓰면, 다른 쓰레드들은 그 값을 볼 수 있습니다. 

<img width="523" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/456718c5-6887-4cf2-9f3c-122733b79aba">

#### CPU and Memory

폰 노이만 구조에서는 CPU가 메모리에서 데이터를 가져와서 실행하고, 메모리는 데이터를 저장하고 있습니다. 멀티코어 CPU 에서는, 2개의 CPU 코어가 하나의 메모리를 공유하는 형태면 될까요? 

<img width="701" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d154071f-f9b8-423a-89c1-886aa826d5b3">

하지만 현실에는, 메인메모리 접근은 느리므로 메모리 계층구조가 존재합니다. 아래와 같이 L1,L2,L3 캐시가 존재합니다. 이 캐시는 메모리보다는 작지만, CPU 에서 훨씬 빠르게 접근할 수 있습니다. 실제 하드웨어에는 레지스터, L1,L2,L3 캐시가 존재합니다. 

<img width="800" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/04dab482-809d-4c43-b366-2e54e6db3315">

<img width="568" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/57da46e8-00dc-4a8d-a4b0-204397382153">

이렇듯 데이터를 저장할 곳은 많습니다. **그러면 CPU 끼리 데이터를 공유할 때, 어디에 있는 데이터를 공유해야 할까요?** 각 계층마다 tradeoff 가 존재합니다. 

실제 구현을 몇가지 봅시다. Dance-hall organization 에서는, 프로세서 내에 L3캐시까지 있고, 칩 바깥에 Interconneted network 에 연결된 하나의 DRAM 이 있습니다. 하지만 이때 CPU 가 메인 메모리에 접근하고, 데이터를 읽어오는 것은 느리기 때문에 하나의 CPU 에서 다른 CPU 의 캐시로 바로 데이터를 직접 전송할 수 있게 하기도 합니다. 


<img width="361" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a27c6206-f6d1-4511-b696-4406bd8246b8">


### Shared memory NUMA

**Shared Memory UMA** 는, uniform memory access 로 모든 메모리 접근이 같은 사이클이 걸리는 것입니다. 반면 **Shared Memory NUMA** 는 Non-uniform memory access 로 메모리 마다 CPU 의 접근 시간이 다른 것입니다.  이때 메모리 접근 latency 는 실제 하드웨어가 어떻게 구현되어 있는지에 따릅니다. 

아래 구조에서, 프로세서0 이 메모리0에 접근하는 시간보다 메모리10에 접근하는 시간이 더 오래 걸립니다. 

<img width="477" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/186af949-cfd2-422d-a5c9-5adba38baa25">

아래의 구조에서, 메모리0의 x를 이용하고, core6 에 돌아가고 있는 쓰레드의 성능을 어떻게 높일 수 있을까요? 첫번째 방법은 x를 메모리1로 할당하는 방법입니다. 두번째는 쓰레드를 core6 에서 core1 로 옮기는 **thread migration** 입니다. 

<img width="451" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4545d8ff-bfea-4d07-999a-85b042e98a61">

하나의 코어에서 다른 코어로 쓰레드를 어떻게 옮길까요? 우선 코어6에서 돌아가고 있는 쓰레드를 멈춘 다음, 컨텍스트를 저장한 후 다시 코어1에서 그 컨텍스트를 가지고 쓰레드를 작동시키면 됩니다. 즉, **context switching** 을 하면 됩니다. 

## C++  threads

C++11 부터 쓰레딩을 지원합니다. (`std: :thread`)  

![carbon (3)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/0155bcb1-c538-400a-9650-44362b2a4241)



- `thread.join()` : caller 는 쓰레드 오브젝트가 끝날 때까지 기다립니다. 이때 block 이 일어납니다. 쓰레드를 싱크하기 위해 사용합니다. 리턴하면, 쓰레드 오브젝트가 소멸됩니다. 
 - `thread.detach()` : detach 를 하게 되면, 실행 중인 쓰레드는 쓰레드 오브젝트에서 분리되며, 서로 다른 것이 됩니다. 쓰레드 오브젝트가 소멸되어도, 분리되어 실행 중인 쓰레드는 영향을 받지 않고 계속 실행할 수 있습니다. 

만약 thread 오브젝트가 아직 "joinable" 한데 소멸시키면 예외가 발생합니다. C++ thread 를 소멸시키려면, `join()` 을 호출하거나(끝난 상태), `detach()` 를 호출해야 합니다. 

#### Create & join

![image](https://github.com/ddoddii/Computer-Science-Study/assets/95014836/419abfd7-60a0-49a9-8289-9dfac5197991)


#### Create & detach

![image](https://github.com/ddoddii/Computer-Science-Study/assets/95014836/43cf0538-3379-4c22-96ab-112a126784da)


#### join() or detach() ?

부모 쓰레드는 프로그래머가 join() 을 호출할 것이라 생각하고, 자식 쓰레드에게 리소스를 할당합니다. 따라서 만약 join() 을 하지 않는다면 detach() 해야 합니다. detach() 하지 않으면 리소스 리크가 생깁니다. 


## Race condition and locks

Counting 프로그램을 예시로 하나 봅시다. 

![carbon (4)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/54ae6cb6-b3f9-40e8-97ff-d456d8807958)


이 프로그램을 멀티 쓰레딩 프로그램으로 만들 수 있을까요?

![carbon (5)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/a5798f5c-de8a-4896-8094-f60ffeae118b)

하지만 결과를 보면, 실제 1000000 보다 적은 결과가 나옵니다. 왜 그럴까요? 이유는 바로 **race condition** 때문입니다. 

어떤 일이 일어나는 지 봅시다.

| Thread0       | count | Thread1       |
| ------------- | ----- | ------------- |
|               | 0     |               |
| Read count(0) |       |               |
|               |       | Read count(0) |
| count++ -> 1  |       |               |
|               |       | count++ -> 1  |
| Write count   | 1     |               |
|               | 1     | Write count   |

쓰레드0,1이 동시에 접근해서 count 값이 0임을 읽습니다. 둘 다 write 하기 전에 read  를 하면, count 값은 여전히 0입니다. 그러나 둘 다 count++ 를 하면, 1이 되고 그러면 2가 write 되는 것이 아니라 1이 write 됩니다. 

### Race condition 

앞에서 봤듯이, read-add-write 연산은 시간이 걸리는 연산들이고 중간에 interrupt 될 수 있습니다. Race condition 이란 결과가 실행의 순서에 따라 영향을 받는 것입니다. 따라서 결과는 매번 실행 때매다 달라집니다. 이것을 다루는 것은 프로그래머의 책임입니다. 

### Mutex

Mutex 는 한 쓰레드/프로세스가 임계 영역에 있으면 다른 쓰레드/프로세스가 못 들어오도록 막습니다. 즉 하난의 쓰레드만이 mutex를 얻어 critical section 내에 있는 공유 자원에 접근할 수 있습니다. 다른 쓰레드들은 mutex 가 unlocked 상태가 될 때까지 기다려야 합니다.  

하지만 동시간대에 하나의 쓰레드만 실행되므로, 프로그램은 무진장 느립니다. 따라서 critical section 의 사이즈를 최소화하는 것도 중요합니다. 

![carbon (6)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/99e1dbb0-027d-497d-a61c-f02fb564b987)


이것을 실행하면, 원래 의도와 같이 1000000 개의 0을 카운트 할 수 있습니다. 여기서 또 주의해야 할 점은, 꼭 `global_mutex.unlock();` 를 해서 뮤텍스를 unlock 해야 합니다. 만약 하지 않는다면, 데드락 상황이 발생할 수 도 있습니다. 

따라서 더 안전하게 뮤텍스를 사용할 수 있는 방법들이 있습니다. 

첫번째는, `lock_guard` 를 사용하는 것입니다. 이것을 사용하면 생성될 때 자동으로 lock을 걸고, 소멸될 때 unlock합니다.

![carbon (8)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/e6fa3b8f-c555-49bd-b362-c69051e9165a)



두번째는 **RAII(Resource Allocation Is Initialization)** 방법 입니다. RAII 는 c++ 에서 자주 사용되는 패턴으로, 리소스가 획득되고 잘 반환되게끔 하는 방법입니다. RAII에선 리소스의 생명주기를 오브젝트의 생명주기와 바인딩합니다. 오브젝트의 초기화 시 리소스를 얻고, 오브젝트가 소멸되면 리소스가 반환됩니다. 

그렇다면 오브젝트가 소멸되었는데도 리소스가 반환되지 않는 경우에는 어떤 것들이 있을까요? 대표적으로 참조 순환이 있습니다. 또한 외부 리소스(파일,네트워크 커넥션) 을 다루는 오브젝트가 소멸 시에 리소스를 반환하지 않으면 리소스가 계속 사용중일 수 있습니다. 

### Deadlock

데드락 상황은 언제 발생하는지 봅시다. swap 연산 때문에 여러 개의 락을 해야 하는 상황일때를 봅시다.

| Thread0    | Thread1    |
| ---------- | ---------- |
| Lock A     | Lock B     |
| Lock B     | Lock A     |
| Wait for B | Wait for A |
| ...        | ...        |

3번째 상황에서 계속 서로가 unlock 하기를 기다리지만, 그런 상황은 오지 않습니다. 따라서 **데드락**이 발생합니다. 

![carbon (9)](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/85702aae-2421-4021-b0e2-d56b96e03dd6)

실제 실행 결과 데드락이 발생합니다. 더 이상 이터레이션을 돌지 못하고 멈춰서는 것을 볼 수 있습니다.
<img width="652" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/e9672023-cc2d-40db-bb97-881573431706">

1. 쓰레드1 이 swap() 을 실행하고 v1.m 에 대한 lock 을 획득합니다.
2. 동시에 쓰레드2도 swap() 을 실행하고 v2.m 에 대한 lock 을 획득합니다.
3. 쓰레드1 는 v2.m 에 대한 lock을 획득하려고 노력하지만, 그 lock 은 이미 쓰레드2가 가지고 있습니다. 따라서 쓰레드1은 lock 이 반환될 때까지 기다립니다.
4. 쓰레드2도 비슷하게 v1.m 에 대한 lock 을 획득하려고 기다리지만, 그 lock 은 이미 쓰레드1이 가지고 있습니다. 쓰레드2도 lock 이 반환될 때까지 기다립니다.


그렇다면 해결책은 무엇일까요?

### Deadlock - 해결책 

가장 간단한 해결책은 여러 개의 뮤텍스를 사용해야 한다면, 항상 같은 순서로 락을 거는 것입니다. `std::lock()`는 여러 개의 뮤텍스를 데드락 없이 락 할 수 있습니다. 

![carbon](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/7bced08c-2085-4dde-8370-2ece7b4b7688)

이렇게 바꾸면 정상적으로 모든 이터레이션을 도는 것을 볼 수 있습니다. 

<img width="197" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/1208a91f-3710-4d85-9264-45519af09efb">

1. 쓰레드1이 `std::lock(v1.m,v2.m)` 을 호출하면 v1.m, v2.m 에 대한 락을 동시에 획득하려 합니다.
2. 만약 두개의 락 중 어느 것이라도 다른 쓰레드가 가지고 있다면, `std::lock()` 는 블럭되고 쓰레드는 기다립니다. 
3. `std::lock()` 는 락들이 일관된 순서로 획득되도록 보장합니다. 따라서 두개의 락을 모두 가진 쓰레드는 임계 영역에 접근하고, 데드락을 피할 수 있습니다. 

<img width="669" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/cc7550f0-e024-43eb-8050-e565ca83f876">

명시적으로 락을 반환하는 것을 신경쓰기 싫으면, `lock_guard` 를 사용해서 락을 관리할 수도 있습니다.

`std::lock_guard<std::mutex> lock_v1(v1.m, std::adopt_lock);` 에서 lock_guard 오브젝트인 lock_v1 을 만들고, v1.m 뮤텍스와 연관시킵니다. `std::adopt_lock` 는 뮤텍스가 이미 락 되어 있다는 것을 나타냅니다(위의 `std::lock(v1.m,v2.m)` 에 의해 락을 획득한 상태). 따라서 lock_v1 이 범위를 벗어나면, v1.m 에 대한 락을 자동으로 반환합니다. 따라서 swap() 함수가 끝나면, lock_v1 과 lock_v2 의 범위를 벗어나고, 자동으로 v1.m , v2.m 에 대한 락을 반환합니다. 

`std::lock()` 과 `std::lock_guard()` 를 같이 사용하면 임계 구역이 끝났을 때 락이 모두 반환되는 것을 보장할 수 있습니다. 


## 쓰레드가 어떤 것을 기다리는 경우

<img width="630" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/2a99f89f-7c45-447b-8682-d3cfb07e66d8">


위와 같은 상황에서, 프로듀서는 공유 큐에 1초마다 아이템을 넣고, 컨슈머는 큐가 비어있지 않는 경우에 아이템을 pop 한다고 합시다. 

### busy wait

최악의 상황을 봅시다. 

![carbon](https://github.com/ddoddii/algorithm/assets/95014836/bfe7bbfb-ba24-4335-aeab-2e48868508b8)

위의 코드에서는 컨슈머가 계속해서 컨디션(큐가 비어 있는지)을 체크합니다. 그 동안 아무것도 하지 않고 리소스를 낭비합니다. 

### busy wait + sleep

<img width="600" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/e21b4bb3-eb81-4a50-8e14-53cfddbd43b9">

컨슈머의 코드만 바꿔보았습니다. 컨슈머는 sleep 하고 주기적으로 깨어납니다. 이때 프로그래머는 여전히 프로듀서가 아이템을 넣는 주기를 알아야 합니다.

### condition variable

새로운 개념인 condition_variable 이 등장했습니다. 이때 `notify_one()` 와 `cond.wait()` 를 사용하면 됩니다. push() 와 pop() 이 일어날 때 락에게 알림을 전송합니다. 그러면 이전처럼 리소스를 낭비하지 않고, 큐가 비어있을 때는 리소스를 반환하여 다른 쓰레드들이 CPU 를 사용할 수 있습니다.

![carbon (2)](https://github.com/ddoddii/algorithm/assets/95014836/62db54eb-6912-4fdc-87af-4784334e966a)

- `cond.wait(unique_lock, predicate function)` 
- `unique_lock` : `lock_guard` 와 비슷하지만 사용자가 unlock 할 수 있습니다. 이때 현재 쓰레드에 의해 락을 했어야 합니다. 
- `notify_one()` : 컨디션 변수를 기다리고 있는 하나의 쓰레드를 깨웁니다. 
- `notify_all()` : 컨디션 변수를 기다리고 있는 모든 쓰레드를 깨웁니다. 

컨슈머의 작동 순서를 자세히 봅시다. 

1. 컨슈머는 `std::unique_lock` 를 뮤텍스 m 에 획득하여, 공유 큐에 대한 접근 권한을 획득합니다. 
2. 그 다음 `cond.wait(lock, [] { return !shared_queue.empty(); })` 를 호출하는데,
	- 컨슈머 쓰레드는 뮤텍스에 대한 락을 반환하고, 컨디션 변수에 알림이 올때까지 기다립니다.
	- 컨디션 변수는 predicate 함수인 `[] { return !shared_queue.empty(); }` 와 연관되어 있는데, 여기서 공유큐가 비어있지 않은지 체크합니다.
	- 프로듀서 쓰레드에 의해 컨슈머의 컨디션 변수에 알림이 오면, 컨슈머 쓰레드는 깨어나서 다시 뮤텍스에 대한 락을 획득합니다. 
	- 공유 큐가 비어있지 않으면 계속 진행하고, 만약 공유큐가 비어있다면 다시 락을 반환하고 다음 알림을 기다립니다. 

### Atomics

![carbon (4)](https://github.com/ddoddii/algorithm/assets/95014836/31a48934-8fdc-4f0a-a474-1d2ce511ea56)


만약 lock-unlock 사이의 작업이 매우 간단하면, `std::atomic` 을 쓸 수 도 있습니다. atomic은 c++ 라이브러리로, 변수에 대한 원자적인 연산을 제공합니다. 원자적 연산은 쪼개질 수 없으므로, 락 없이도 쓰레드 안전성을 보장합니다. `std::atomic` 은 변수에 대한 read, write 연산이 원자적으로 이루어짐을 보장하며, 이것은 race condition 가 일어나지 않음을 보장합니다. 

atomic 을 사용하면, 뮤텍스 락보다는 살짝 빠르지만 여전히 보통 연산들에 비해서는 현저히 느립니다. 

### Barrier

Barrier도 동기화 메카니즘입니다. 모든 쓰레드가 barrier 에 도착할 때까지 쓰레드들이 기다리는 동기화 포인트로 작동합니다. 모든 쓰레드들이 barrier 에 도착하면, 동시에 실행합니다. join() 과 비슷하지만, join()과 달리 쓰레드들을 종료시키지는 않습니다. c++ 20에서 기본 라이브러리로 포함되었습니다. 

![carbon (5)](https://github.com/ddoddii/algorithm/assets/95014836/2695692a-e0d3-4913-a7e3-b4af6c03ae5b)

위에서는 뮤텍스(`mutex_`)와 컨디션 변수(`cv_`)를 사용해 Barrier를 구현했습니다. 

`Wait()` 함수 내에서:
1. 쓰레드는 뮤텍스에 대해 unique_lock 을 획득합니다. 
2.  `count_` 를 1씩 감소시키며, barrier 에 도달한 쓰레드의 개수를 셉니다. 
3. `count_` 가 0이 되면, barrier 에 모든 쓰레드들이 도달했다는 의미입니다. 이 경우 count 는 원래 count 값 (`initial_count_`)으로 초기화 되며, 기다리고 있는 쓰레드들은 `cv_.notify_all()` 을 이용해 알림을 전송합니다. 
4.  `count_` 가 0이 아니면, 쓰레드는 `cv_.wait()` 를 사용하여 컨디션 변수에 기다립니다. 쓰레드는 `count_ == initial_count_`  가 참이 될때까지 블럭됩니다. 

### Thread-safety

**Thread-safe** 하다는 것은 코드에서 공유 데이터를 조작하는데도 여러 개의 쓰레드를 가지고 코드를 실행해도 괜찮다는 의미입니다.  하지만 대부분의 자료구조들은 thread-safe 하지 않습니다. 아주 간단하게, lock 을 걸어서 thread-safe 하게 만들 수 있습니다. 하지만 이런 경우 굉장히 느려집니다. 어떻게 하면 thread-safe 하면서 속도를 많이 늦추지 않을 수 있을까요?

아래 **Linked-list queue** 예시를 봅시다. 

![carbon (6)](https://github.com/ddoddii/algorithm/assets/95014836/2bd2ced0-35f2-4691-b90d-152208a5e16d)

push() 는 tail 만 사용하고, pop() 는 head 만 사용하는데, 모든 것에 락을 걸 필요가 있을까요? 변경할 노드(head, tail) 에만 lock 을 걸도록 구현해봅시다. 

![carbon (11)](https://github.com/ddoddii/algorithm/assets/95014836/6edf7bf5-f452-41ac-9ac8-076146373e66)



이렇게 구현해도, push() 와 pop() 이 동시에 일어날 수 있습니다. 이러한 방법을 **fine-grained locking** 이라고 합니다. 하지만 이 경우에도 **데드락** 상황이 발생할 수는 있습니다. 

pop() 에서, head 노드와 new_head 노드의 락을 획득하려 합니다. 이 상황에서 만약 2개 쓰레드가 다른 순서로 락을 획득하면 데드락이 발생할 수 있습니다. 
1. 쓰레드A가 pop() 함수를 실행하고, head 노드의 락을 획득합니다.
2. 쓰레드B도 pop() 함수를 실행하고, 쓰레드A 이후 head 노드의 락을 획득합니다.
3. 쓰레드A는 다음 노드(new_head)로 움직이고, 락을 획득하려 합니다.
4. 쓰레드B도 다음 노드(new_head)로 움직이고, 락을 획득하려 합니다.

이 상황에서 쓰레드A 와 쓰레드B 는 서로 락을 반환하기를 기다리면서 데드락이 발생합니다. 따라서 락을 구현할 때 이러한 점들을 신경써야 합니다. 



## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- https://stackoverflow.com/questions/37015775/what-is-different-between-join-and-detach-for-multi-threading-in-c
- https://en.cppreference.com/w/cpp/thread/lock_tag
- https://stackoverflow.com/questions/27089434/whats-the-difference-between-first-locking-and-creating-a-lock-guardadopt-lock