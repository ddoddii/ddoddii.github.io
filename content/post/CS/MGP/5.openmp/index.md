---
title: "멀티쓰레딩을 편리하게 해주는 OpenMP 사용법"
date: 2024-04-17
draft: false
summary : "OpenMP 101"
tags: ["Multicore-GPU-Programming"]
categories : ["CS"]
series: ["Multicore-GPU-Programming"]
series_order: 5
slug : "OpenMP"
toc : true
katex : true
markup: 'mmark'
---

## OpenMP - why?

지금까지 개별적인 쓰레드를 이용한 병렬프로그래밍 방법들을 봤습니다. **OpenMP** 는 코드에 따른 적절한 수의 쓰레드를 만들고, 스케쥴 해주어 코드를 자동으로 병렬화 시켜줍니다.  

![carbon](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fd04d206-ceb1-4285-a0fd-3b0087cc8908)


OpenMP 는 실제 어셈블리 명령어를 만드는 것이 아니라, 컴파일러에게 정보를 주는 방식으로 작동합니다. 병렬화가 쉽고, 쉬운 문법을 가지고 있지만 컴파일러가 어떠한 작업을 할지 확실하지 않을 때는 원하든 결과가 나오지 않을 수 도 있습니다. 

## Directives

`#pragma omp parallel {...}` 은 프로그래머와 컴파일러 사이의 인터페이스 처럼 작동합니다. 이것을 사용해 병렬화가 필요한 지역을 지정할 수 있습니다. {} 내에 있는 지역은 fork-join 을 사용하여 병렬화됩니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/524196bc-00dd-434d-8adc-33e0e9da9f40)


## Critical section

임계구역은 보통 아래 처럼 설정할 수 있습니다.

```cpp
m.lock();
// critical section...
m.unlock();
```

OpenMp 를 사용하면, 아래와 같이 설정할 수 있습니다.

```cpp
#pragma omp critical
{
// critical section...
}
```

그럼 병렬처리와 임계 구역을 동시에 설정해야 할때는 어떻게 해야할까요?

```cpp
a();
#pragma omp parallel
{
	c(1);
	#pragma omp critical
	{
		c(2); //critical section
	}
	c(3);
	c(4);
}
z();
```


![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/770aa507-b44c-4071-9947-fdfb7053383a)


## Shared/private

### Shared

`std::threads` 에서는 함수 안에 있는 변수는 private 이었습니다. OpenMP 에선, shared 가 기본값입니다. 

```cpp
#include <omp.h>
#include <cstdio>
#include <cassert>
#include <stdlib.h>

int main(int argc, char **argv)
{
	int shared_int = -1;
	omp_set_num_threads(2);
	#pragma omp parallel
	{
		//shared_int in shared
		int tid = omp_get_thread_num();
		printf("Thread ID %2d  | shared_int = %d\n",tid,shared_int);
	}
...
}
```

만약 `shared_int` 변수를 수정하면, **race condition** 이 생깁니다. 

### Private

```cpp
#include <omp.h>
#include <cstdio>
#include <cassert>
#include <stdlib.h>

int main(int argc, char **argv)
{
	omp_set_num_threads(2);
	int is_private = -2;
	
	#pragma omp parallel private(is_private)
	{
		int tid = omp_get_thread_num();
		printf("Thread ID %2d  | is_private(before) = %d\n",tid,is_private);
		is_private = tid();
		printf("Thread ID %2d  | is_private(after) = %d\n",tid,is_private);
		assert(is_private==tid);
	}
	printf("Main thread  | is_private = %d\n", is_private);

	return 0;
}
```

```text
Thread ID 1  | is_private (before) = 32763
Thread ID 1  | is_private (after) = 1
Thread ID 0  | is_private(before) = 0
Thread ID 0  | is_private (after) = 0
Main thread  | is_private = -2
```

`is_private` 변수가 각 쓰레드마다 할당되었음을 볼 수 있습니다. 하지만 똑같이 초기화 된 것은 아닙니다. {} 안에서 `is_private` 변수를 수정해도 블럭 바깥에는 영향을 미치지 않습니다. 

### First private

Firstprivate 를 사용하면 모든 로컬 변수가 초기화됩니다. 

```cpp
#include <omp.h>
#include <cstdio>
#include <cassert>
#include <stdlib.h>

int main(int argc, char **argv)
{
	omp_set_num_threads(2);
	int is_private = -2;
	
	#pragma omp parallel firstprivate(is_private)
	{
		int tid = omp_get_thread_num();
		printf("Thread ID %2d  | is_private(before) = %d\n",tid,is_private);
		is_private = tid();
		printf("Thread ID %2d  | is_private(after) = %d\n",tid,is_private);
		assert(is_private==tid);
	}
	printf("Main thread  | is_private = %d\n", is_private);

	return 0;
}
```


```text
Thread ID 1  | is_private (before) = -2
Thread ID 1  | is_private (after) = 1
Thread ID 0  | is_private(before) = -2
Thread ID 0  | is_private (after) = 0
Main thread  | is_private = -2
```

범위 바깥에 `is_private` 변수를 -2로 초기화한것이 그래도 적용됨을 볼 수 있습니다. 그러나 여전히 블럭 내에서  `is_private` 변수를 수정하는 것은 블럭 바깥에 영향을 미치지 않습니다. 

### Last private

```cpp
#include <omp.h>
#include <cstdio>
#include <cassert>
#include <stdlib.h>

int main(int argc, char **argv)
{
	omp_set_num_threads(2);
	int last_private = -2;
	
	#pragma omp parallel for lastprivate(last_private)
	for (int i=0;i<10;i++)
	{
		int tid = omp_get_thread_num();
		printf("Thread ID %2d excuting i=%d  | last_private(before) = %d\n",tid,i,is_private);
		is_private = tid();
		printf("Thread ID %2d excuting i=%d  | last_private(after) = %d\n",tid,i,is_private);
		assert(last_private==i);
	}
	printf("Main thread  | last_private = %d\n", last_private);

	return 0;
}
```

Lastprivate 는 이터레이션 내 마지막으로 수정된 값으로 고정됩니다. 그리고 private 과 같이 초기화되지 않습니다. 

```text
Thread ID 1 executing i=5 | last_private (before) = 1704464448
Thread ID 1 executing i=5 | last_private(after) = 5
Thread ID 1 executing i=6 | last_private (before) = 5
Thread ID 1 executing i=6 | last_private (after) = 6
Thread ID 1 executing i=7 | last_private(before) = 6
Thread ID 1 executing i=7 | last_private (after) = 7
Thread ID 1 executing i=8 | last_private (before) = 7
Thread ID 1 executing i=8 | last_private(after) = 8
Thread ID 1 executing i=9 | last_private(before) = 8
Thread ID 1 executing i=9 | last_private(after) = 9 
Thread ID 0 executing i=0 | last_private (before) = 0
Thread ID 0 executing i=0 | last_private (after) =0
Thread ID 0 executing i=1 | last_private(before) = 0
Thread ID 0 executing i=1 | last_private(after) = 1
Thread ID 0 executing i=2 | last_private(before) = 1
Thread ID 0 executing i=2 | last_private (after) = 2
Thread ID 0 executing i=3 | last_private (before) = 2
Thread ID 0 executing i=3 | last_private(after) = 3
Thread ID 0 executing i=4 | last_private(before) = 3
Thread ID 0 executing i=4 | last_private(after) = 4
Main thread | last_private = 9
```

## Sections

병렬화를 위한 다른 **sections** 를 제공할 수 도 있습니다. 

```cpp
omp_set_num_threads(10);

#pragma omp parallel sections
{
	#pragma omp parallel section
	for(int i=0;i<1000;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section A\n",tid);
	}

	#pragma omp parallel section
	for(int i=0;i<1000;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section B\n",tid);
	}
}

```

## Single

만약 하나의 블럭이 하나의 쓰레드로만 실행하려고 하면, single 을 사용할 수 있습니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/eed368e8-d92b-44b2-aef1-140b291b16c8)

```cpp
#pragma omp parallel 
{
	#pragma omp single
	for(int i=0;i<10;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section A\n",tid);
	}
	// Implicit barrier !

	#pragma omp for
	for(int i=0;i<100;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section B\n",tid);
	}
}
```

```text
Thread ID 2 section A
Thread ID 2 section A
Thread ID 2 section A
Thread ID 2 section A
Thread ID 2 section A
...
Thread ID 9 section B
Thread ID 9 section B
Thread ID 2 section B
Thread ID 0 section B
Thread ID 8 section B
...
```

## Barriers

OpenMP 는 블럭 끝에 implicit barrier 를 가지고 있습니다. 

```cpp
a();
#pragma omp parallel 
{
	b();
	#pragma omp for
	for (int i=0;i<10;++i){
		c(i);
	}
	d();
}
z();
```

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/e735e3a5-f2c7-4f72-8ec0-a521f219509b)

만약 implicit barrier 를 원하지 않으면, nowait 을 사용하면 됩니다.


```cpp
a();
#pragma omp parallel 
{
	b();
	#pragma omp for nowait
	for (int i=0;i<10;++i){
		c(i);
	}
	d();
}
z();
```

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/bb95798c-ecfc-4b4d-b507-91a4566b104e)

만약 명시적으로 barrier 를 사용하고 싶다면, `#pragma omp barrier` 를 설정하면 됩니다. 

## Master

section 과 비슷하지만, master 쓰레드에 의해 실행되는 것을 보장합니다. 

```cpp
#pragma omp parallel 
{
	#pragma omp master
	for(int i=0;i<100;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section A\n",tid);
	}

	#pragma omp for
	for(int i=0;i<100;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section B\n",tid);
	}
}
```

이렇게 하면 Thread ID 가 0인 master 쓰레드만 section A를 실행합니다.

```text
Thread ID 2 section B
Thread ID 2 section B
Thread ID 5 section B
Thread ID 5 section B
Thread ID 3 section B
Thread ID 0 section A
Thread ID 3 section B
Thread ID 4 section B
Thread ID 4 section B
Thread ID 0 section A
Thread ID 0 section A
...
```

**Master + explicit barrier** 를 같이 쓰면, single 과 비슷하게 작동합니다. 

```cpp
#pragma omp parallel 
{
	#pragma omp master
	for(int i=0;i<100;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section A\n",tid);
	}
	
	#pragma omp barrier
	#pragma omp for
	for(int i=0;i<100;i++){
		int tid = omp_get_thread_num();
		printf("Thread ID %2d section B\n",tid);
	}
}
```

```text
Thread ID 0 section A
Thread ID 0 section A
Thread ID 0 section A
Thread ID 0 section A
Thread ID 0 section A
Thread ID 0 section A
....

Thread ID 7 section B
Thread ID 7 section B
Thread ID 9 section B
Thread ID 0 section B
Thread ID 0 section B
Thread ID 0 section B
Thread ID 1 section B
Thread ID 1 section B
...
```

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/c32c2879-5ddc-4dff-a6ce-61092ad434d9)

## Reduction


```cpp
int main(int argc, char **argv)
{
	int sum=0;
	#pragma omp parallel for
	for (int i=0;i<100;i++) {
		sum += i;
	}
	printf("Sum : %d\n", sum);
	return 0;
}
```

이 경우에, 답은 엉뚱한 값이 나옵니다. 어떻게 하면 정확한 결과를 얻을까요?

```cpp
int main(int argc, char **argv)
{
	int sum=0;
	#pragma omp parallel for reduction(+:sum)
	for (int i=0;i<100;i++) {
		sum += i;
	}
	printf("Sum : %d\n", sum);
	return 0;
}
```

reduction 을 사용하면 local sum 을 만들고, global sum에 더해줍니다. 

## Summary

OpenMP 를 사용하면, 쓰레드를 직접 만들었을 때보다 간편하게 병렬 프로그래밍을 할 수 있지만, 어디서 틀렸는지 정확히 모를 수 도 있다는 단점이 있습니다. 따라서 정확하게 문법을 숙지하여 openmp 를 사용해야 합니다. 




## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- https://engineering.purdue.edu/~smidkiff/ece563/files/ECE563OpenMPTutorial.pdf
- https://www.openmp.org/