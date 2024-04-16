---
title: "멀티쓰레드에서 쓰레드 간 작업을 어떻게 균일하게 분할할까?"
date: 2024-03-25
draft: false
summary : "workload balancing, thread pool"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 3
slug : "thread-management"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

본 포스팅에서는 **"성능"** 에 집중해 보겠습니다. 어떻게 하면 여러 개의 쓰레드에 태스크를 균일하게 분배하고, 태스크를 분할하는 방법들에 대해 알아봅시다. 

## Workload balancing

워크로드를 분배하는 것은 성능에 큰 영향을 미칩니다. 아무리 작은 양일지라도 **로드가 불균등하게 분배되면 성능에 치명적인 악영향**을 미칠 수 있습니다. 

작업을 할당하는 방법에는 2가지(static,dynamic) 이 있습니다. 정적 할당은 컴파일 타임에 작업의 할당이 결정되고, 동적 할당은 런타임 동안에도 작업의 할당이 결정됩니다. 

### Static assignment

행렬A 에서 sum(A) 를 계산한다고 합시다. 행렬의 행마다 요소의 개수는 동일하므로, 쓰레드마다 같은 아이템을 할당해도 충분합니다. 

그러나 항상 워크로드 양이 같은 것은 아닙니다. 인풋값에 따라 워크로드가 달라질 때는 동적 할당이 필요합니다. 

### Dynamic assignment

> granularity : breaking down of larger tasks into smaller ones.

워크가 쌓여 있고, worker 가 와서 워크(work)를 하나씩 가져갑니다. 그럼 전체 프로그램을 어떻게 나눠서 할당해야 할까요? 만약 하나의 유닛을 너무 **작게** 할당하면, 태스크를 할당하고 관리하는데 오버헤드가 듭니다. 또한 태스크 사이에 스위칭 하는데 임계구역에 있는 시간이 늘어납니다. 

<img width="355" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/ff8a533e-afcd-4292-8917-698ee199fc32">


반면 하나의 유닛을 너무 **크게** 할당하면, 태스크가 워커들 사이에 고르게 분배되지 않을 수 있습니다. 

<img width="354" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/92e1652f-2ccd-40e8-b189-f135efbd37b7">

Granularity 를 위한 일반적인 규칙들을 봅시다. 
- 쓰레드(혹은 프로세서)의 수보다 태스크 수가 많을 때 : 불균형이 발생합니다.
- 쓰레드(혹은 프로세서)의 수보다 태스크 수가 적을 때 : 임계구역에 있는 시간이 늘어납니다.
- 따라서 가장 실용적인 방법은 직접 측정해보는 것입니다 !

### Scheduling
- **Static** 
	 - 이터레이션은 `block_size` 의 블럭들로 나누어지고, 쓰레드에게 정적으로 할당됩니다.
- **Dynamic**
	 - 쓰레드가 하나의 블럭을 끝내면, 다음 블럭이 할당됩니다. 이터레이션 안에 work 가 불규칙할 때 유용합니다. 
- **Guided**
	 - 다이나믹 스케쥴링 방법이지만, 시간이 지남에 따라 블럭 사이즈는 줄어듭니다. 이터레이션 당 work 가 규칙적이지만 쓰레드가 다른 시간 대에 도착하는 경우 유용합니다. 초반에는 큰 블럭을 할당해서 스위칭 오버헤드를 최소화하고, 마지막에는 작은 블럭을 할당해서 불균형을 줄입니다. 
	 - 하지만 이 경우 실행 시간을 예측해야 하기 때문에 preprocessor 가 필요하다는 문제점이 있습니다. 


    <img width="500" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/7ad018c5-ec01-4d98-bd01-52c79b6caea5">


## Thread pool (work queue)

쓰레드 풀은, 미리 여러 개의 쓰레드를 만들어 놓는 것입니다. 모든 workload 가 큐안에 있으면, 각 쓰레드는 큐에서 workload 를 할당받을 수 있습니다. 이 방법은 로드 밸런싱에 효과적입니다. 

간단한 예시를 봅시다. 만약에 트리구조에서, 루트에서 시작해 탐색하고 싶다고 합시다. 각 노드마다 work 가 할당되어 있습니다.(=각 노드마다 어떠한 일을 수행해야 합니다.) 그리고 트리의 구조는 인풋 데이터에 의해 결정됩니다. 

workload 는 트리에 있는 노드 수 로 결정됩지만, 코드를 작성하는 시점에서는 얼만큼의 workload 가 생길지 예측할 수 없습니다. 


#### sol1 - spawn threads

첫번째 방법은 쓰레드를 계속해서 만드는 방법입니다. 하지만 쓰레드를 계속해서 만들면 컨텍스트 스위칭으로 인해 오버헤드가 발생합니다. 

#### sol2 - thread pool

work 를 큐에 넣습니다. 그리고 고정된 수의 쓰레드들을 쓰레드 풀에 넣습니다. 각 쓰레드는 큐로부터 작업을 받고, 큐로 작업을 push 합니다.

<img width="611" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/370de5bd-5a8a-4186-8513-a8be5312f93a">

이제 4개의 쓰레드로 작업할 수 있습니다. 하지만 workload 큐 여러 개의 쓰레드가 공유하므로, thread-safe 하게 만들어야 합니다. 따라서 여기에 임계 구역을 만들어야 합니다. 

#### sol3 - Thread-local work queue

위에서의 모든 쓰레드들이 하나의 큐를 공유하면 동시에 read/write 할 때 문제가 발생할 수 있습니다. 따라서 로컬 큐를 사용하자는 해결책이 나왔습니다. 

<img width="417" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/c190cc8a-b945-4610-9cbd-686ed108d6e5">

여기서는 임계 구역이 없습니다. 쓰레드는 각자의 큐로만 푸쉬하고 작업을 가져옵니다. 따라서 임계 구역, 동기화에 드는 오버헤드가 없습니다. 하지만 이 방법은, 정적할당 방식과 마찬가지로 큐 사이에 불균형이 생길 수 있습니다.

따라서 이 방법에 대한 해결책으로 **steal, spill** 이 나왔습니다. 

<img width="416" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/66916e27-a1bb-4a20-a4a6-f755934d4f9e">

**Work stealing** 은 자신의 쓰레드 로컬 큐에 아무것도 작업이 없을 때, 다른 큐에서 작업을 가져오는 것입니다. 이때 각 큐는 하나의 프로듀서가 있고, 여러 개의 컨슈머가 있는 형태입니다. 

<img width="412" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/a04e5e75-77c4-446e-8f96-a4fafd59c425">


**Work spilling** 은 자신의 쓰레드 로컬 큐에 너무 많은 작업이 있으면 다른 큐에 작업을 보내는 것입니다. 이때 각 큐는 여러 개의 프로듀서가 있고, 하나의 컨슈머가 있는 형태입니다. 

|                                     | Stealing           | Spilling                 |
| ----------------------------------- | ------------------ | ------------------------ |
| Queue reader                        | Multiple           | Single                   |
| Queue writer                        | Single             | Multiple                 |
| Who pays the cost of moving the job | Idle thread        | Overloaded thread        |
| Good balance when..                 | A few idle threads | A few overloaded threads |

## Amdahl's Law

암달의 법칙은, **프로그램은 병렬처리가 가능한 부분과 불가능한 순차적인 부분으로 구성되므로 프로세서를 아무리 병렬화 시켜도 더 이상 성능이 향상되지 않는 한계가 존재 한다는 법칙** 입니다. 

어플리케이션은 2가지 부분으로 나눌 수 있습니다 :
 - Serial part(S) - 병렬처리가 불가능한 부분 
	 - 덧셈, 나누기 ..
 - Parallel part(P) - 병렬처리가 가능한 부분
	 - 벡터 덧셈, 행렬 곱 ...

- \\(T(1) = T_{s}+ T_{p}\\)-> 코어 1개일 때
- \\(T(P) = T_{s} + \frac{T_p}{P}\\)-> 코어가 P개 일 때 (Serial 부분은 단축시킬 수 없고, Parallel 부분만 코어의 개수만큼 나누어 단축시킬 수 있다.)
- \\(Speedup \ S = \frac{T(1)}{T(P)} =\frac{T_{s}+ T_{p}}{T_{s}+\frac{T_p}{P}}\\)  
- \\(f=fraction \ of \ the \ sequential \ portion\\)
- \\(S(p) < \frac{1}{f+\frac{(1-f)}{p}}\\)

위의 수식으로 봐도, Speedup 은 한계가 있습니다. 아래 예시들을 통해 알아봅시다.

예시1) 만약 40%가 sequential 이고, 60%가 parallel 일 때, 6개의 프로세서가 있으면 얼만큼의 speedup 을 할 수 있을까요?

 \\(S=\frac{T(1)}{T(P)}=\frac{1}{f+\frac{(1-f)}{p}}=\frac{1}{0.4+\frac{0.6}{6}}=2\\)

예시2) 만약 50%가 sequential 이고, 50%가 parallel 일 때, 2배의 speedup 을 하려면 몇개의 프로세서가 필요할까요?

- \\(f=0.5\\)
- \\(2 = \frac{1}{0.5+\frac{0.5}{P}}\\)
- \\(P=\infty\\)-> 프로세서의 수를 아무리 늘려도, sequential 비율이 높으면 2배의 성능 증가는 불가능합니다. 



## Reference

- Multicore and GPU Programming, 연세대학교 박영준 교수님
- https://medium.com/distributed-knowledge/optimizations-for-c-multi-threaded-programs-33284dee5e9c
- https://www.codewithc.com/c-and-thread-pools-balancing-load-in-multi-threading/
- https://matgomes.com/thread-pools-cpp-with-queues/
- https://modoocode.com/285