---
title: "Basic Parallel Architectures에 대해 알아보자"
date: 2024-03-13
draft: false
summary : "Superscalar, Multi-core, Vector Processor"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 1
slug : "basic-parallel-architecture"
toc : true
katex : true
markup: 'mmark'
---
{{< katex >}}


본 포스팅에서는 기본적인 **Parallel Architectures** 3가지에 대해 살펴보겠습니다. 

## Superscalar processors (SISD)

**Superscalar processor** 는 1개의 프로세서를 가지고, **instruction-level parallelism(ILP)** 을 통해 병렬성을 구현하는 형태입니다. Scalar Processor 과는 다르게, Superscalar processor는 클록사이클에 1개 이상의 명령어들 실행할 수 있는데, 이것은 프로세서의 실행 유닛에 여러 개의 명령어를 할당함으로써 가능합니다. 여기서 각 실행 유닛은 다른 프로세서가 아니라, arithmetic logic unit(ALU) 와 같은 싱글 CPU 내에 있는 실행 리소스 입니다. 

### ILP는 무엇인가요?

명령어들은 연속적으로 작성됩니다. 하지만 모든 명령어가 꼭 순차적으로 실행될 필요는 없습니다. 만약 명령어간 독립적이라면, 그들은 병렬적으로 실행될 수 있습니다. Superscalar processor는 독립적인 명령어들을 찾고, 자동적으로 그들을 병렬적으로 실행합니다. 

아래의 코드를 봅시다. 

```
a = x*x + y*y + z*z
```

이것을 어셈블리어로 표현하면, 아래와 같습니다.

```assembly
// r0=x, r1=y, r2=z, r3=a
(1) mul r0, r0, r0
(2) mul r1, r1, r1
(3) mul r2, r2, r2
(4) add r0, r0, r1
(5) add r3, r0, r2
```

프로세서는, 어셈블리어 명령을 순차적으로 실행하는 기계입니다. Program Counter(PC) 는 다음 실행할 명령어의 위치를 가리킵니다. 위의 어셈블리어는 총 5개의 사이클이 걸립니다.

5개의 명령어들이 있지만, 모두가 서로 독립적인 것은 아닙니다. 이것을 data dependency 가 있다고 표현합니다. (4) 의 add 는 (1), (2) 의 연산이 끝난 후 실행되어야 합니다. 즉 모든 연산이 한번에 실행될 수 없습니다. ALU가 무한개가 있더라고, 3번의 사이클이 필요합니다. 

<img width="297" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f61b464d-8f82-4480-a2ef-8a0cf05c2528">

이때 3번의 사이클에 끝낼 수 있도록 필요한 최소 ALU 개수는 몇개일까요? 위 그림에서 (3) 연산을 Cycle 1로 미루면 2개의 ALU 만으로도 모든 연산을 3개 cycle 안에 끝낼 수 있습니다. 

그렇다면 이제 Scalar vs. Superscaler 에 대해 비교해보겠습니다. 

### Scalar vs. Superscalar

**Scalar processor** 는 명령어들이 파이프라인으로 실행되기는 하지만, 하나의 사이클 내에는 오직 한개의 명령어가 fetched / decoded 될 수 있습니다. 


**Superscaler processor** 는 여러 개의 병렬적인 명령어 파이프라인을 가질 수 있습니다. 2-way super scalar processor 는 사이클 당 2개의 명령어를 fetch 할 수 있습니다. 


<img width="563" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fd7dc043-89ce-450f-ad1c-8debaa873555">

여기서 주의해야 할 점은 Superscalar 도 여전히 1개의 싱글 CPU 라는 점입니다. 

<img width="512" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/207523a6-15bc-492b-b8af-48f3bd361135">

위 그래프를 보면 Superscalar 파이프라인을 이용하면 더 많은 명령어들을 더 짧은 시간 내에 처리할 수 있음을 볼 수 있습니다. 


Superscalar 는 프로그래머가 코드를 재작성할 필요 없이, 프로세서가 독립적인 명령어들을 찾고, **자동으로** 이들을 병렬적으로 실행한다고 했습니다. 굉장히 좋아보이는데,  그렇다면 Superscalar는 만능인걸까요? 😲

하지만 여기서 맹점은, 모든 프로그램은 **data dependency** 를 가지고 있다는 점입니다. 따라서 실행 유닛을 증가시켜서 속도를 증가시키는데에는 한계가 있습니다. 

<img width="422" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/97d86ff9-bc52-46c8-b21d-f535cff160be">

## Multi-core processors (MIMD)

이제 Multi-core processor 가 등장합니다. 얘는 아예 코어가 여러 개입니다. 

<img width="684" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/742bb274-8e15-4d73-9755-142f22e0ddde">

그러나 멀티코어 프로세서에서 각 코어는 대체로 싱글 코어 프로세서보다 느립니다. 하지만 2개의 코어가 있다면, 각 코어가 싱글 코어의 80% clock frequency 를 가지고 있다고 해도 2* 0.8 = 1.6 이므로, 여전히 싱글코어보다는 빠릅니다. 

그렇다면 아까 예시 코드를 다시 봅시다. 

```assembly
// r0=x, r1=y, r2=z, r3=a
(1) mul r0, r0, r0
(2) mul r1, r1, r1
(3) mul r2, r2, r2
(4) add r0, r0, r1
(5) add r3, r0, r2
```

이 코드를 어떻게 멀티코어 프로세서에서 실행할까요? 🤔 

정답은.. 못합니다. 멀티코어 프로세서의 이점을 활용하기 위해서는 프로그래머가 직접 코드를 재작성해야 합니다. 

우리는 이때 **쓰레드**를 사용합니다. 아래는 subroutine 과 thread 의 차이입니다. 쓰레드가 subroutine 을 호출하면, 제어권이 그 subroutine 의 쓰레드로 넘어갑니다. 이 방법은 여전히 싱글 코어만 사용합니다. 

반면 쓰레드가 다른 쓰레드를 호출하면, 기존의 오리지널 쓰레드와  별개로, 제어권을 가진 쓰레드가 생깁니다. 이 쓰레드들은 병렬적으로 실행 가능하고, 기존의 오리지널 쓰레드가 종료되어도 계속 실행됩니다. 이렇게 멀티쓰레드를 사용하면 여러 개의 코어를 모두 사용할 수 있습니다. 

<img width="820" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a832b4cd-e4fd-4a46-a70d-07b0543443f4">

### 쓰레드는 무엇인가요? 

그렇다면 **쓰레드**는 정확히 무엇일까요?  

 **쓰레드**는 프로세스의 경량화된 버전으로, 프로세스 내에서 작업의 실행 단위입니다. 쓰레드들 끼리는 code,data,heap 등 몇가지의 리소스를 공유합니다. 하지만 쓰레드만의 고유한 영역인 pc, stack, registers 도 있습니다. 

<img width="431" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ab7b2349-efc3-491c-bff5-399a42540a5b">

왜 쓰레드는 **코드영역**과 **데이터 영역**을 공유할까요? 프로세스의 **코드영역**에는 실제 수행되는 코드가 컴파일 되어 CPU 명령어가 바이너리 형태로 저장되어 있습니다. 두개의 쓰레드는 같은 프로그램 내에 속해있기 때문에, 수행하는 명령어들의 집합은 같습니다. 프로세스의 **데이터 영역**은 전역 변수나 static 변수 등 프로세스 시작과 함께 생성되는 데이터가 저장되어 있습니다. 


왜 **program counter, stack, register** 는 공유할 수 없을까요? Program Counter 는 다음 실행할 명령어를 가리키는데, 각 쓰레드는 같은 프로그램이지만 각각 다른 명령어를 실행하기 때문입니다. **스택**에는 로컬 변수, 함수 호출 기록, 로컬 변수가 저장되는데  쓰레드의 실행 경로에 따른 상태 정보를 유지해야 하기 때문에, 쓰레드별로 스택을 가집니다. 따라서 각 쓰레드 별로 같은 이름의 로컬 변수가 있더라도, 힙 영역 내에 다른 데이터를 할당받을 수 있습니다. 

그렇다면, 만약 프로세서에서 사이클마다 모든 쓰레드에게 같은 명령어를 실행하게 한다면, pc 를 공유할 수 있을까요? 이런 특수한 상황에서는 가능합니다. 그렇다면 이때는 스택은 공유할 수 있나요? 아닙니다. 여전히 각 쓰레드가 다른 데이터를 다루기 때문에 공유할 수 없습니다. 

예시 코드를 하나 더 봅시다. 

```cpp
#include <iostream>

const int N = 100;
int main()
{
	int a[N], b[N], k[N], c[N];

	for (int i = 0; i < N; i++)
	{
		a[i] = i;	
		b[i] = 1000 * i;
		k[i] = 10;
		c[i] = 0;
	}

	for (int i = 0; i < N; i++)
	{
		c[i] = k[i] * a[i];
		c[i] += k[i] * b[i];
	}
	return 0;
}
```

위의 코드를 멀티쓰레딩을 적용하면 아래와 같이 작성할 수 있습니다.

```cpp
#include <iostream>
#include <thread>
#include <vector>

int *a, *b, *k, *c;

void mac(int tid, int num_threads)
{
	for (int i = 0; i < N / num_threads; i++)
	{
		int idx = tid * (N / num_threads) + i;
		c[idx] = k[idx] * a[idx];
		c[idx] += k[idx] * b[idx];
	}
	return;
}

int main(int argc, char *argv[])
{
	...
	std::vector<std::thread> threads;
	
	for (int t = 0; t < NT; t++)
	{
		threads.push_back(std::thread(mac, t, NT)); //create multiple thread
	}
	
	for (auto& thread : threads)
	{
		thread.join();
	}
	
	return 0;
}
```

2개의 코어가 작업을 절반씩 하도록 하게 합니다. `mac(0,2)` 는 0번 쓰레드에, `mac(1,2)` 는 1번 쓰레드에 작업을 시킬 수 있습니다. 코드는 동일하지만, Tid 가 다르므로 각각의 쓰레드는 다른 작업을 하고 있습니다. 

<img width="727" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/067a0f01-8dbc-4a00-b9a8-1ba9a35336d8">

core0 을 이용해 A,B,K 의 앞의 절반을 사용해 C의 절반을 만들고, core1 을 이용해 A,B,K 의 뒤의 절반을 사용해 C의 나머지 절반을 계산할 수 있습니다. 

Loop 들은 병렬 처리를 하기 효과적입니다. 왜냐하면 루프의 각 이터레이션은 다른 데이터를 가지고 동일한 작업을 하기 때문입니다. 이때 `#pragma omp parallel for` 를 작성하면,  C++ 의 OpenMP 라이브러리는 쓰레딩이 된 코드를 자동으로 작성해주기도 합니다. 

이렇게 데이터를 분할해서 병렬로 실행하면... 모든 문제가 해결된 것일까요? 🤔 

위의 상황에서는, 모든 코어에서 fetch / decode 명령어를 실행하고 있습니다. 꼭 그럴 필요가 있을까요? 없습니다. 즉, 명령어를 가져오는 것을 한번만 해도 됩니다. 왜냐하면 **사이클마다 같은 명령어를 실행하기 때문**입니다. 

이때 **한개의 코어에서만 fetch/decode 를 실행**하면, 실행 리소스를 줄여 성능을 조금이나마 개선할 수 있습니다. 

## Vector Processing

이제 **Vector Processing** 이 등장합니다. 여기서는 하나의 명령어 스트림만 있고, 여러개의 ALU 가 있는 구조입니다. 그리고 여러개의 ALU가 하나의 fetch/decode 유닛을 공유합니다. 

<img width="186" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/94007376-3229-4408-9927-48ae3f1ad717">

이때 데이터의 여러 조각을 가지고 한번에 연산을 수행합니다. 각 ALU 는 같은 명령어를 수행하지만, 각각 다른 데이터를 가지고 연산합니다. 

그렇다면 이때 아래와 같은 **조건문**은 어떻게 실행할까요?

```cpp
if (A[i] < 0) {
 // do something..
}
else {
 // do something else...
}
```

**마스킹**을 통해 실행할 수 있습니다. 

<img width="700" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/579f3a1b-86f0-4a1b-97df-c546fd0c934d">

마스킹을 통해서, 조건을 만족하는 ALU만 연산을 수행하게 하는 것입니다.  컴파일러는 masking vector 를 만들어서, 조건에 맞는 ALU를 실행하고 다른 것은 꺼버립니다. 
ALU 를 모두 사용하지 않아서 약간의 효율성 문제가 있으나, 여전히 모든 ALU 가 같은 명령어에 대한 연산을 하게 할 수 있다는 장점이 있습니다. 

## Summary

지금까지 여러 개의 기본적인 병렬 처리 구조에 대해 살펴봤습니다. 용어를 다시 한번 정리해보겠습니다. 

<span style="font-size:120%">**SISD : Single Instruction, Single Data**</span>  
- Scalar Processor
- SuperScalar Processor

<span style="font-size:120%">**MIMD : Multiple Instruction, Multiple Data**</span>  

- Multi-core Processor

<span style="font-size:120%">**SIMD :  Single instruction, multiple data**</span>  

-  Vector processor

<span style="font-size:120%">**MISD: Multiple instruction, single data**</span>  

- Systolic Array
- Google TPU
- And many other NPUs (Neural processing units)

SuperScalar Processor 과 Multicore Processor 의 차이점은 무엇일까요? SuperScalar 는 여러 개의 명령어를 실행하지만, 여전히 프로그램의 입장에서 봤을 때는 싱글 쓰레드에서 실행합니다. 반면 Multicore는 두개의 쓰레드로 명령어들을 실행하는 형태입니다. 

<img width="462" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/29f1c85d-a38a-49ea-a528-2d7f247f1dc0">



## Reference

- Multicore and GPU Programming (2024-1), 연세대학교 박영준 교수님
- [Computer Architecture: What is the difference between scalar and superscalar?](https://www.quora.com/Computer-Architecture-What-is-the-difference-between-scalar-and-superscalar)
- https://www.researchgate.net/figure/Superscalar-versus-scalar-clock-cycles_fig2_283345112
- https://w3.cs.jmu.edu/kirkpams/OpenCSF/Books/csf/html/ProcVThreads.html#:~:text=Since%20all%20threads%20in%20a,also%20lead%20to%20some%20disadvantages.
- https://www.quora.com/Do-different-processes-share-the-heap-In-physical-memory-is-there-any-specific-area-for-heap











