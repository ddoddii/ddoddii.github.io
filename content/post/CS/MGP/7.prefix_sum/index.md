---
title: "Prefix Sum : 효율적인 연산을 위한 가이드"
date: 2024-05-01
draft: false
summary : "prefix sum 톺아보기"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 7
slug : "prefix-sum"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}

## Intro

병렬 컴퓨팅과 데이터 처리의 세계에서는 효율적인 알고리즘이 성능 최적화를 위해 매우 중요합니다. 본 포스팅에서는 데이터 처리 작업의 속도와 효율성을 높이는 강력한 기술인 **prefix sum**의 핵심 개념을 다룹니다. 기본적인 reduce 연산에 대한 이해를 시작으로, 병렬 reduce로의 전환을 탐구한 후, prefix sum과 그 병렬 버전에 대해 깊이 있게 다룹니다. 특히 Kogge-Stone 알고리즘과 Brent-Kung 알고리즘에 중점을 두어 그 중요성과 현대 컴퓨팅에서의 응용을 강조합니다.

## Reduce

### Sum reduce problem

sum 을 구하는 프로그램의 예시를 봅시다.

{{< codeimporter url="https://raw.githubusercontent.com/ddoddii/Multicore-GPU-Programming/master/prefix-sum/sum_single.cpp" type="cpp" >}}



이 작업을 병렬화하고 싶을 때는 어떻게 할까요? 아래처럼 멀티 쓰레드로 하면 됩니다. 

{{< codeimporter url="https://raw.githubusercontent.com/ddoddii/Multicore-GPU-Programming/master/prefix-sum/sum_parallel.cpp" type="cpp" >}}

하지만 결과를 보면 **race condition** 이 생겼음을 알 수 있습니다. 


<img width="303" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/b90113a0-bc0e-41a0-a249-d4f677bb71a6">

<img width="277" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/fbec4dbc-f4db-482a-93eb-25d7a55bc069">

#### Try 1 : locks

고치는 첫번째 방법은 **lock** 을 사용하는 것입니다. 이 방법은 커다란 성능 저하를 일으킵니다.

```cpp
std::mutex global_mutex;

void worker(int *input, int size, int *output)
{
    for (int i = 0; i < size; ++i)
    {
        global_mutex.lock();
        *output += input[i];
        global_mutex.unlock();
    }
}
```

그런데 앞서서 짧은 연산에는 lock 보다 atomic 이 낫다고 배웠습니다. 

#### Try 2 : atomics

```cpp
void worker(int *input, int size, std::atomic<int>* output)
{
    for (int i = 0; i < size; ++i)
    {
        *output += input[i];
    }
}
```

lock 보다는 성능 저하가 작지만, 여전히 atomic 도 느린 연산입니다. 

다른 방법은 없을까요? 만약 무한대의 코어를 가지고 있다고 하면, 가장 최대로 병렬화할 수 있는 방법은 무엇일까요?

### Serial reduce

만약 코어가 여러 개 있어도 순차적 연산이면 쓸모가 없습니다. 즉, 병렬화될 수 없습니다. 아래와 같이 순차적인 덧셈이 있는 연산이라고 할 때 \\(O(N)\\) 의 시간이 걸립니다. 

<img width="621" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/17e7f403-d5cb-4212-8e09-244863fb1429">



## Parallel reduce

병렬화를 하고 싶다면 병렬화가 가능하도록 코드를 작성해야 합니다. 이렇게 된다면 무한개의 코어가 있을 때 연산 시간은 \\(O(logN)\\) 으로 단축될 수 있습니다. 

<img width="578" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/98920b64-a0f5-4de8-aaef-f61a17a2eff7">

이제 유한개의 코어가 있을 때 병렬화를 어떻게 해야할지 알아봅시다. 

## Parallel prefix sum

<img width="571" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/21a47dbe-1990-41da-8560-c2e577b1697e">

**Prefix Sum**은 배열의 각 요소에 대해 이전 요소들의 합을 계산하는 방식입니다. (이 방법은 코딩테스트에서도 많이 사용됩니다.)

Prefix Sum을 사용하면 배열의 일부 구간에서 대한 합을 매우 빠르게 구할 수 있게 해줍니다. N개의 원소로 이루어진 배열이 주어졌을 때 반복문을 통해 부분 배열의 합을 구하려면 O(N)이 걸리는데, 누적합을 이용하면 부분합을 0(N+M)으로 구할 수 있습니다.  

병렬 Prefix Sum의 알고리즘에는 **Kogge-Stone 알고리즘**과 **Brent-Kung 알고리즘**이 있습니다.

### Kogge-Stone algorithm

초기 배열 A = {3,1,7,0,4,1,6,3} 이라고 합시다. 

Kogge-Stone 알고리즘은 여러 단계로 나누어 수행됩니다. 각 단계에서는 이전 단계에서 계산된 값을 사용하여 새로운 값을 계산합니다. 아래는 각 단계의 일반적인 과정입니다:

- 단계 k에서, 각 프로세서는 거리 2^k 만큼 떨어진 이전 값을 더합니다.
- 이 과정을 통해 모든 요소는 자신의 이전 요소들과의 합을 계산하게 됩니다.

<img width="516" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/3799f0fb-4cc1-4b4d-907f-19e658e14e1e">


cpp 구현은 [여기](https://github.com/ddoddii/Multicore-GPU-Programming/blob/master/prefix-sum/kogge-stone.cpp) 있습니다. 

만약 **무한대의 코어**가 있다면 병렬처리가 최대한으로 이루어져 각 연산 단계가 동시에 수행될 수 있어 \\(O(logN)\\) 의 시간이 걸립니다. 예를 들어, 입력 배열의 크기가 8인 경우 최대 3단계 안에 연산이 완료됩니다.

그러나 우리는 **유힌개의 코어**만 가지고 있습니다.🥲 첫 단계에서는 N-1 연산, 두번째에서는 N-2, ... 마지막 단계에서는 N-N/2 번의 연산이 일어나 전체 연산량은 \\(O(NlogN)\\) 입니다. P개의 코어를 사용한다면 연산량은 \\(O(\frac{N}{P} logN)\\) 이 됩니다. 그리고 중간 결과를 저장하기 위해 추가적인 메모리가 필요합니다. 단계마다 새로운 배열을 생성해야 하므로, 총 \\(O(log N)\\) 배의 메모리가 추가로 필요합니다.

이러한 문제를 해결하기 위해 **double buffering** 를 사용하기도 합니다. 매번 새로운 배열을 생성하는 것보다 2개의 버퍼에 중간 결과를 저장하는 방법입니다. 이 방법으로 2배 이상의 메모리 공간을 절약할 수 있습니다.

<img width="412" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/2dce244e-83f3-4c9e-be3d-282f10b25e06">


### Brent-Kung algorithm

Brent-Kung 알고리즘은 balanced binary tree 를 사용합니다. (실제 트리 구조는 아니고, 연산 패턴을 차용합니다.)

두 단계로 구성되는데, **upsweep 단계**와 **downsweep 단계**입니다. 

1. Phase1 : upsweep

배열을 재귀적으로 절반씩 나누어 합을 구합니다. 중간 결과를 트리 구조로 저장합니다.

<img width="573" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/8908a397-bc55-4e79-9c00-64f2ed735554">

<img width="552" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/78261c9c-1c5d-4305-be4d-d9f2c99f6f85">

2. Phase2 : downsweep

이 단계에서는 중간 결과를 사용하여 최종 prefix sum을 계산합니다. 이 과정에서 다시 트리 구조를 따라가며 값을 계산합니다.

<img width="655" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/5ac07726-7750-40ea-8795-f3c0b9944111">

<img width="535" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/e823b07b-1eac-41e6-9326-74b6729afabd">


예시를 통해 자세히 봅시다. 

1. **초기화**

- 입력 배열: A = [a0, a1, a2, a3, a4, a5, a6, a7]
- 출력 배열: S = [s0, s1, s2, s3, s4, s5, s6, s7]

1. **Upsweep 단계**

- 레벨0 : 각 요소는 그대로 유지됩니다.

    S0: [a0, a1, a2, a3, a4, a5, a6, a7]

- 레벨1: 각 페어의 합을 계산합니다.

    S1: [a0, (a0 + a1), a2, (a2 + a3), a4, (a4 + a5), a6, (a6 + a7)]

- 레벨2 : 레벨 1에서 계산된 값을 사용하여 더 큰 페어의 합을 계산합니다.

    S2: [a0, (a0 + a1), (a0 + a1 + a2 + a3), a3, a4, (a4 + a5), (a4 + a5 + a6 + a7), a7]

- 레벨3 : 최종 합을 계산합니다.

    S3: [a0, (a0 + a1), (a0 + a1 + a2 + a3), (a0 + a1 + a2 + a3 + a4 + a5 + a6 + a7), a4, (a4 + a5), (a4 + a5 + a6 + a7), a7]

3. **Downsweep 단계**

- 레벨3 : 최종 합을 루트 노드로 유지하고 나머지 값들을 그대로 유지합니다.

    S3: [a0, (a0 + a1), (a0 + a1 + a2 + a3), 0, a4, (a4 + a5), (a4 + a5 + a6 + a7), 0]

- 레벨2 : 중간 결과를 사용하여 아래로 내려가면서 값을 계산합니다.

    S2: [a0, (a0 + a1), 0, (a0 + a1 + a2 + a3), a4, (a4 + a5), 0, (a4 + a5 + a6 + a7)]

- 레벨1 : 레벨 2에서 계산된 값을 사용하여 최종 프리픽스 합을 계산합니다.

    S1: [a0, 0, (a0 + a1), (a0 + a1 + a2 + a3), a4, 0, (a4 + a5), (a4 + a5 + a6 + a7)]

- 레벨0 : 최종 프리픽스 합을 계산합니다.

    S0: [0, a0, (a0 + a1), (a0 + a1 + a2), (a0 + a1 + a2 + a3), (a0 + a1 + a2 + a3 + a4), (a0 + a1 + a2 + a3 + a4 + a5), (a0 + a1 + a2 + a3 + a4 + a5 + a6)]

Brent-Kung 알고리즘은 새로운 메모리 공간이 필요 없습니다. 시간복잡도를 생각해보면, 두 단계에서 각각 \\(logN\\) 연산이 필요합니다(총 \\(2logN\\)). 따라서 시간복잡도는 \\(logN\\) 입니다. 연산량은 각 단계에서 N-1개의 연산이 필요하므로, \\(O(N)\\) 으로 표현할 수 있습니다. P개의 프로세서가 있다면 \\(O(N/P)\\) 가 걸릴 것입니다.

#### 두 알고리즘의 비교

||Kogge-Stone|Brent-Kung|
|---|-------|-----|
|Time Complexity|\\(LogN\\)|\\(2logN\\)|
|Computational Complexity|\\(O(NlogN)\\)|\\(O(N)\\)|
|Speedup with P cores|\\(O(\frac{N}{P}logN)\\)|\\(O(N/P)\\)|


## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- https://people.cs.vt.edu/yongcao/teaching/cs5234/spring2013/slides/Lecture10.pdf
- https://www.geeksforgeeks.org/prefix-sum-array-implementation-applications-competitive-programming/
- https://en.wikipedia.org/wiki/Prefix_sum
- https://junstar92.tistory.com/260
- Programming Massively Parallel Processors, ch.9 