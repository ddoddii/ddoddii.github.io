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
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(2);
    int is_private = -2;

/*
private : not initialized
Modifying is_private within parallel block does not modify the value outside block
*/
#pragma omp parallel private(is_private)
    {
        int tid = omp_get_thread_num();
        printf("Thread ID %2d  | is_private(before) = %d\n", tid, is_private);
        is_private = tid;
        printf("Thread ID %2d  | is_private(after) = %d\n", tid, is_private);
        assert(is_private == tid);
    }
    printf("Main thread  | is_private = %d\n", is_private);

    return 0;
}

```

<img width="380" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5fbcca7a-7758-4523-95a1-f7f40574b161">

`is_private` 변수가 각 쓰레드마다 할당되었음을 볼 수 있습니다. 하지만 똑같이 초기화 된 것은 아닙니다. {} 안에서 `is_private` 변수를 수정해도 블럭 바깥에는 영향을 미치지 않습니다. 

### First private

Firstprivate 를 사용하면 모든 로컬 변수가 초기화됩니다. 

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(2);
    int is_private = -2;
/*
firstprivate : initialized to value outside region
Modifying is_private within parallel block does not modify the value outside the block
*/
#pragma omp parallel firstprivate(is_private)
    {
        int tid = omp_get_thread_num();
        printf("Thread ID %2d  | is_private(before) = %d\n", tid, is_private);
        is_private = tid;
        printf("Thread ID %2d  | is_private(after) = %d\n", tid, is_private);
        assert(is_private == tid);
    }
    printf("Main thread  | is_private = %d\n", is_private);

    return 0;
}
```

<img width="679" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ee9952ed-6ff5-4cf7-9e8d-b2f0551c507e">



범위 바깥에 `is_private` 변수를 -2로 초기화한것이 그래도 적용됨을 볼 수 있습니다. 그러나 여전히 블럭 내에서  `is_private` 변수를 수정하는 것은 블럭 바깥에 영향을 미치지 않습니다. 

### Last private

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(2);
    int last_private = -2;
/*
lastprivate : not initialized,
becomes what's written in the last iteration (no matter when, which thread executed it!)
*/
#pragma omp parallel for lastprivate(last_private)
    for (int i = 0; i < 10; i++)
    {
        int tid = omp_get_thread_num();
        printf("Thread ID %2d excuting i=%d  | last_private(before) = %d\n", tid, i, last_private);
        last_private = i;
        printf("Thread ID %2d excuting i=%d  | last_private(after) = %d\n", tid, i, last_private);
        assert(last_private == i);
    }
    printf("Main thread  | last_private = %d\n", last_private);

    return 0;
}

```

**Lastprivate** 는 이터레이션 내 마지막으로 수정된 값으로 고정됩니다. 그리고 private 과 같이 초기화되지 않습니다. 

<img width="695" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9a2445fa-17f9-4b9c-900e-fe690fca69c3">

### FirstPrivate & LastPrivate

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(2);
    int last_private = -2;
/*
lastprivate & firstprivate : initialized,
becomes what's written in the last iteration (no matter when, which thread executed it!)
*/
#pragma omp parallel for firstprivate(last_private) lastprivate(last_private)
    for (int i = 0; i < 10; i++)
    {
        int tid = omp_get_thread_num();
        printf("Thread ID %2d excuting i=%d  | last_private(before) = %d\n", tid, i, last_private);
        last_private = i;
        printf("Thread ID %2d excuting i=%d  | last_private(after) = %d\n", tid, i, last_private);
        assert(last_private == i);
    }
    printf("Main thread  | last_private = %d\n", last_private);

    return 0;
}
```

firstpriavate, lastprivate 같이 사용할 수 도 있습니다.

<img width="732" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4f57fbac-7b54-403c-b8bc-cbdb5cb552bd">

## Sections

병렬화를 위한 다른 **sections** 를 제공할 수 도 있습니다. 

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(10);

#pragma omp parallel sections
    {
#pragma omp section
        for (int i = 0; i < 10; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section A\n", tid);
        }

#pragma omp section
        for (int i = 0; i < 10; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section B\n", tid);
        }
    }

    return 0;
}
```

<img width="528" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5188d2e1-c25e-4014-bad4-b0e56745d829">


## Single

만약 하나의 블럭이 하나의 쓰레드로만 실행하려고 하면, single 을 사용할 수 있습니다. 

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/eed368e8-d92b-44b2-aef1-140b291b16c8)

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(10);

#pragma omp parallel
    {
#pragma omp single
        for (int i = 0; i < 10; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section A\n", tid);
        } // implicit barrier !!

#pragma omp for
        for (int i = 0; i < 100; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section B\n", tid);
        }
    }

    return 0;
}
```

<img width="478" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/30186494-f5fc-46d8-a5c6-00b53071e43a">

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

<img width="466" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/944a02f8-d997-424d-8923-0ff1caade744">


**Master + explicit barrier** 를 같이 쓰면, single 과 비슷하게 작동합니다. 

```cpp
#include <cassert>
#include <cstdio>
#include <omp.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(10);

#pragma omp parallel
    {
/*
master + explicit barrier works like "single"
*/
#pragma omp master
        for (int i = 0; i < 5; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section A\n", tid);
        }
#pragma omp barrier
#pragma omp fpr
        for (int i = 0; i < 5; i++)
        {
            int tid = omp_get_thread_num();
            printf("Thread ID %2d section B\n", tid);
        }
    }

    return 0;
}
```

<img width="667" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ea86c4f3-2355-425a-938e-e6b66270da9f">

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

<img width="584" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a7c924ca-030a-480b-b322-afc0df2074cf">

이 경우에, 답은 엉뚱한 값이 나옵니다. 어떻게 하면 정확한 결과를 얻을까요?

```cpp
#include <iostream>
#include <omp.h>

int main(int argc, char **argv)
{
    omp_set_num_threads(10);
    int sum = 0;

#pragma omp parallel for reduction(+ : sum)
    for (int i = 0; i < 100; i++)
    {
        sum += i;
    }
    printf("Sum : %d\n", sum);
    return 0;
}
```

**reduction** 을 사용하면 local sum 을 만들고, global sum에 더해줍니다. 0부터 99까지 합인 4950이 올바르게 출력되는 것을 볼 수 있습니다. 

<img width="554" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/adc69f68-495e-4a7b-9845-981a08a7f392">

## Nested loop

```cpp
omp_set_num_threads(4);
a();
#pragma omp parallel for
for (int i=0;i<4;i++) {
	for (int j=0;j<4;j++) {
		c(i,j);
	}
}
z();
```

`parallel for` 는 가장 바깥쪽 루프만 병렬화합니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/01496a52-bf90-4cde-ad3b-953bd683093a)

```cpp
omp_set_num_threads(4);
a();
#pragma omp parallel for
for (int i=0;i<3;i++) {
	for (int j=0;j<6;j++) {
		c(i,j);
	}
}
z();
```


만약 가장 바깥 쪽 루프의 이터레이션 수보다 쓰레드의 수가 많으면 어떻게 될까요? 아래 처럼 쓰레드 한개가 idle 한 상황이 발생합니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/61636455-c7cc-4348-8f7a-f466f2a24f7d)

이 상황을 해결하는 방법들을 봅시다.

<span style="font-size:120%">**Bad Way(1)**</span>

```cpp
omp_set_num_threads(4);
a();
for (int i=0;i<3;i++) {
	#pragma omp parallel for
	for (int j=0;j<6;j++) {
		c(i,j);
	}
}
z();
```

안쪽 루프를 병렬화하는 방법입니다. 가끔 해결책이 될 수 있지만, 항상 해결책이 되지는 않습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f4584222-fe28-4f17-92c3-213c355da930)

<span style="font-size:120%">**Bad Way(2)**</span>

```cpp
omp_set_num_threads(4);
a();
#pragma omp parallel for
for (int i=0;i<3;i++) {
	#pragma omp parallel for
	for (int j=0;j<6;j++) {
		c(i,j);
	}
}
z();
```

이렇게 안쪽, 바깥쪽 루프에 모드 병렬화를 하면 두번째 pragma 는 무시됩니다. 따라서 맨 위에만 pragma 를 붙인 것과 같은 결과입니다.

<span style="font-size:120%">**Good way(1)**</span>

```cpp
a();
#pragma omp parallel for
for (int ij = 0; ij < 3; ++ij) {
	c(ij/6, ij%6);
}
z();
```

중첩된 루프를 하나의 루프로 푸는 방식입니다. 따라서 총 18번의 이터레이션이 생깁니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0bc37cab-cc29-4690-a334-01cc839e5c25)

<span style="font-size:120%">**Good way(2)**</span>

위의 방식을 자동으로 해주는 pragma 의 `collapse` 를 사용할 수 있습니다. 

```cpp
omp_set_num_threads(4);
a();
#pragma omp parallel for collapse(2)
for (int i=0;i<3;i++) {
	for (int j=0;j<6;j++) {
		c(i,j);
	}
}
z();
```

## Summary

OpenMP 를 사용하면, 쓰레드를 직접 만들었을 때보다 간편하게 병렬 프로그래밍을 할 수 있지만, 어디서 틀렸는지 정확히 모를 수 도 있다는 단점이 있습니다. 따라서 정확하게 문법을 숙지하여 openmp 를 사용해야 합니다.

작성한 실제 코드는 아래 레포지토리의 **openmp 폴더**에서 볼 수 있습니다.


{{< github repo="ddoddii/Multicore-GPU-Programming" >}}

## Reference
- Multicore and GPU Programming, 연세대학교 박영준 교수님
- https://engineering.purdue.edu/~smidkiff/ece563/files/ECE563OpenMPTutorial.pdf
- https://www.openmp.org/