+++
author = "Soeun"
title = "Garbage Collector 작동 방식"
date = "2024-05-09"
summary = "자바와 파이썬의 Garbage Collector로 알아본 GC 작동원리"
categories = [
    "CS"
]
tags = [
    "Java", "Python", "Garbage-Collector"
]
+++

본 포스팅에서는 자바의 Garbage Collector 작동방식에 대해 알아보고 나아가 파이썬의 Garbage collector 과 비교하고자 합니다. 

C/C++ 의  malloc, calloc, realloc, free 와 달리 자바와 파이썬에서는 메모리를 자동으로 관리해주는 **Garbage Collector** 가 존재합니다. 

Garbage Collector 가 무엇이고, 언제 동작하고, 어떻게 동작하는지 차근차근 알아가 봅시다. 

## GC는 왜 존재할까?

앞서 언급했던 것처럼, 자바와 파이썬 같은 고수준의 언어에서는 프로그래머가 직접 메모리를 관리하지 않아도 되게끔 메모리 관리를 GC가 대신 해줍니다. 그렇다면 애초에 왜 메모리를 관리해야 할까요? 

흔히 GC가 메모리를 대신 관리해주는 장점으로는 메모리 릭이 언급됩니다. 그러나 스택오버플로우의 [이 답변](https://stackoverflow.com/questions/1695131/why-is-garbage-collection-so-important) 을 보면, 더 중요한 이유는 **메모리 안정성(memory-safety)** 때문입니다. C/C++ 에서 하는 것처럼 개발자가 직접 메모리 할당을 관리하면, 여러가지 오류들이 발생할 수 있습니다. 

예를 들면, 사용중인 객체에 대한 메모리를 너무 빨리 해제하는 경우가 있습니다. 이 경우에는 디버그 하는 것도 굉장히 어려워지는데, 왜냐하면 해제된 메모리를 다시 재할당하기 어려울 수 도 있기 때문입니다. 이러한 오류들은 GC가 메모리를 관리하는 환경에서는 발생하지 않습니다.

## GC에서 사용하는 전략들

### Tracing

[Tracing](https://en.wikipedia.org/wiki/Tracing_garbage_collection) 은 GC에서 가장 흔하게 사용하는 방법입니다. Tracing은 root object 에서 참조하는 객체들을 계속 타고 가면서 객체가 접근가능한지(reachable) 또는 접근 불가능한지(unreachable) 판단합니다. 구체적으로, **reachability** 는 2가지 방법으로 판단합니다.
1. root object 로부터 접근 가능한 객체들. 이 객체들은 call stack(현재 호출되는 함수의 로컬 변수와 파라미터가 저장되는 곳) 으로부터 참조되거나, global variable들이 있습니다. 
2. 접근가능한 객체가 참조하는 객체는 reachable 합니다. 

이 unreachable 들은 GC의 수거 대상이 됩니다.  이 방식을 구현하는 알고리즘에는 여러가지가 있습니다. 

### Reference Counting

[Reference Counting](https://en.wikipedia.org/wiki/Reference_counting)은 참조되는 횟수를 이용하는 방법입니다. 참조되는 횟수가 0인 객체는 즉시 메모리가 해제됩니다. 굉장히 간단하기 때문에 구현하기도 쉽습니다. 

그러나 Reference Counting 은 여러 가지 문제점들이 있습니다. 
-  **Cycles**

	  만약 2개 이상의 객체가 서로 참조한다면, 이것은 서로 참조하는 **순환 참조**를 만들 수 있습니다. 그렇게 되면 참조하는 횟수는 영원히 0이 되지 않습니다. 따라서 Reference Counting 을 사용하는 여러 언어, 특히 CPython 에서는 순환 참조 문제를 해결하기 위해 cycle-detecting 알고리즘을 사용하기도 합니다. 다른 해결 방법은 [weak reference](https://en.wikipedia.org/wiki/Weak_reference) 를 사용하는 방법입니다. weak reference에 의해서만 참조되는 객체는 weakly reachable 로 판단되고, GC의 수거 대상이 됩니다. 
- **Space overhead (reference count)**

	  Reference Counting 에서는 객체의 참조 횟수를 저장하기 위한 추가적인 공간이 필요합니다. 참조횟수는 대체로 unsigned pointer 를 사용해서 나타내는데, 이것은 32 또는 64bits 가 추가적으로 사용됩니다. 
- **Speed overhead (increment/decrement)** 

	  멀티쓰레드 환경에선, 참조 횟수를 증가/감소 시킬 때 race condition 이 발생할 수 있습니다. 따라서 참조 횟수를 증가/감소 시키는 연산은 원자적 연산이어야 합니다. 이러한 원자적 연산에는 [compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap) 이 있습니다. 

GC에서 자주 사용하는 전략들을 봤으니, 우선 자바에서 어떠한 형태로 GC를 구현했는지 알아봅시다. 


## 자바에서의 GC

자바의 GC를 이해하기 전에, [JVM 의 런타임 데이터 영역](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5)에 대한 이해가 선행되어야 합니다. 자바에서 사용하는 메모리 영역에는 PC 레지스터, JVM 스택, 힙(Heap), 메서드 영역, 런타임 상수 풀, 네이티브 메서드 스택이 있습니다. 이 중 GC가 발생하는 부분은 **힙 영역**입니다. 

### JVM의 Heap 영역

그러면 **힙(Heap) 영역**에 대해 상세히 알아봅시다. 힙 영역에는 클래스 인스턴스와 배열이 저장됩니다.  자바의 GC는 '**weak generational hypothesis**' 를 기반으로 하는데, 여기에는 2가지 전제가 있습니다. 
- 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.
- 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다. 
이러한 가설을 십분 활용하기 위해 HotSpot VM에서는 힙 영역을 크게 Young, Old, Perm 으로 물리적 공간을 나누었습니다. Perm 영역은 거의 사용되지 않기 때문에 배제하겠습니다. Young 영역은 다시 eden 영역과 2개의 Survivor 영역으로 나뉩니다. 따라서 고려해야 할 메모리 영역은 총 4개(eden, survivor1, survivor2, old) 입니다. 


<img width="713" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/786eaef5-578b-41aa-9331-bd0de95e3faf">


일단 메모리에 객체가 생성되면, eden 영역에 객체가 지정됩니다. Eden 영역에 데이터가 꽉 차면, survivor0 또는 survivor1 로 데이터가 옮겨가게 됩니다. 이 때 2개의 survivor 영역 중 1개는 반드시 비어 있어야 합니다. 할당된 survivor 영역이 차면, GC가 작동하며 eden 영역에 있는 객체와 꽉 찬 survivor 영역에 있는 객체가 비어있는 survivor  영역으로 이동합니다. 이러한 작업들을 반복하며, survivor0,1 을 왔다 갔다 하던 객체들은 Old 영역으로 이동합니다. 

바로 Young 영역에서 Old 영역으로 객체가 옮겨가는 경우도 있는데, 이것은 객체의 사이즈가 너무 커서 survivor 영역에 맞지 않을 때 입니다. 

### GC의 종류 

GC는 크게 2가지 타입으로 나뉘는데, minor GC와 major GC입니다. 
- **minor GC** : Young 영역에서 발생하는 GC
- **major GC** : Old 영역이나 Perm 영역에서 발생하는 GC 

GC가 발생하거나 객체가 다른 영역으로 이동할 때 병목현상이 발생할 수 도 있는데, 이를 해결하기 위해 위해 Hot Spot JVM은 **TLABs(Thread-Local Allocation Buffers)** 를 사용합니다. TLAB는 스레드 당 할당되는 Eden 영역의 장은 덩어리인데, 각 쓰레드에는 자기가 갖고 있는 TLAB에만 접근할 수 있기 때문에,  다른 쓰레드에 영향을 주지 않고 메모리를 할당할 수 있습니다. 


#### <span style="font-size:130%">**5가지 GC 방식**</span>

JDK 에서 사용하는 GC 방식에는 크게 6가지가 있습니다. 
- Serial Collector
- Parallel Collector
- Parallel Compacting Collector
- Concurrent Mark-Sweep Collector(CMS)
- Garbage First Collector (G1)
- Z Garbage Collector(ZGC) 

WAS 나 자바 어플리케이션을 실행 할 때, 6가지 방식 중 하나를 지정해서 선택할 수 있습니다. Java 17 의 기본 GC는 ZGC 입니다. 

### Serial Collector

이 방식은 Young 영역과 Old 영역이 순차적으로 처리되며 하나의 CPU를 사용합니다. 

Young 영역에서는 앞서 설명했던 방식대로, 
1. 살아있는 객체들은 eden 영역에 있습니다.
2. eden 영역이 꽉 차게 되면 survivor0 영역으로 살아있는 객체가 이동합니다. 이때 survivor 영역에 들어가기 너무 큰 객체는 바로 Old 영역으로 이동합니다. survivor0 영역이 꽉 차면 survivor1 영역으로 이동합니다. 
3. survivor1 영역이 꽉 차면, eden 영역이나 survivor0에 남아있는 객체들은 Old 영역으로 이동합니다. 

<img width="379" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b422733e-6b45-4dcd-b9a2-8848ee2e6408">

이후 Old 영역에서는 **Mark-Sweep-Compact** 알고리즘을 사용합니다. 이 알고리즘은, 가비지 컬렉션 대상 객체를 식별(Mark) 하고, 제거(Sweep) 하며 객체가 제거되어 파편화된 메모리 영역을 앞에서부터 채워나가는 작업(Compaction) 을 수행합니다. 

<img width="734" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dce0331d-6d03-4d5c-80dc-7ef24bd598eb">

- Mark : Old 영역으로 이동된 객체들 중 살아있는 객체를 식별합니다. 
- Sweep : Old 영역의 객체들을 훑는 작업을 수행하며 쓰레기 객체를 식별합니다. 
- Compact : 필요 없는 객체들을 지우고 살아있는 객체들을 한 곳으로 모읍니다. 

### Parallel Collector

이 방식은 Serial Collector과 달리 Young 영역에서 콜렉션을 병렬로 처리합니다. Old 영역에서는 Serial Collector과 마찬가지로 Mark-sweep-compact 를 사용합니다. 

### Concurrent Mark-Sweep Collector(CMS)

이 방식은 low-latency collector 로 알려져 있으며, 힙 메모리 영역의 크기가 클 때 적합합니다. **Young 영역**에 대한 GC는 Parallel Collector 과 동일합니다. 

**Old 영역**에서는 아래와 같은 단게를 거칩니다.

1. **Initial Mark** : 매우 짧은 대기 시간으로 살아 있는 객체를 찾는 단계입니다.  root 로부터 시작해서 스택 영역을 훑어 메서드에서 참조되는 것들만 등록합니다. 쉽게 생각하면 탐색할 범위를 한정시키는 것입니다. 
2. **Concurrent Mark** : 서버 수행과 동시에 살아 있는 객체에 표시를 해놓는 단계입니다. 초기 마크 이후 추가 참조를 찾는 작업입니다. 
3. **Remark** : Concurrent 표시 단계에서 표시하는 동안 변경된 객체에 대해서 다시 표시하는 단계입니다. Concurrent Mark 가 서버 실행 중에 수행되므로, 그동안 수정된 것들을 다시 마킹합니다. 
4. **Concurrent Sweep** : 서버 수행과 동시에 표시되어 있는 쓰레기를 정리하는 단계입니다. 

Initial Mark, Remark 에서는 **Stop-the-World** 상태로, GC을 실행하기 위해 JVM이 애플리케이션 실행을 멈춥니다. 하지만 Concurrent Mark, Concurrent Sweep 에서는 어플리케이션 실행과 같이 수행됩니다. 결국 **단계를 나누어 놓은 이유는 STW 상태에 빠져 있는 시간을 최소화 하기 위함**입니다. 


<img width="552" alt="image" src="https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/bb3027bb-0865-49ac-81f8-cbacc8d1a29c">

CMS의 원래 의도는, 톰캣을 띄우는 서블릿 컨테이너나 어플리케이션 컨텍스트에서 IoC 컨테이너 같은 객체들은 어플리케이션이 처음에 시작하고 죽기까지 계속 살아있는 객체라서 Old 영역에 들어갈 것이라고 가정을 했습니다. 그리고 요청이 들어올 때마다 새로운 스레드를 할당하면 생기는 객체들은 Young 영역에 있을 것이라 가정했습니다. 그래서 목적 자체가 Old 영역에는 오래된 객체들이 있고 Young 영역에는 금방 생성된 객체들이 있어서, Young 영역에는 객체들이 있는 동안 빨리 지우면 Old 영역으로 넘어가는 객체들은 별로 없고 **GC가 Young 영역만 돌아가게 하는 것**이었습니다. 

그런데 문제는, **Young 영역의 크기가 너무 작을 때** 생깁니다. GC는 Young 영역이 일정 퍼센트 이상 차면 돌아갑니다. 만약 GC가 3초마다 작동하고, 어떤 요청 처리 시간이 10초라고 가정하면 요청이 처리되는 동안 3번의 GC가 작동하고, 이때마다 GC는 해당 객체가 살아있으니 오래 살아있다고 판단하고 Survivor 영역으로 넘기고 또 Old 영역으로 옮깁니다. 이런 상황이 누적되면 Old 영역도 꽉 차게 됩니다. 그렇게 되면 JVM 은 어쩔 수 없이 full STW 를 유발하는 Parallel GC 로 돌아가게 됩니다. 

또 다른 문제점은, CMS는 굉장히 오래전에 고안된 방법으로, 과거에는 인프라에 메모리 영역이 그리 크지 않았습니다. 그러다 보니 JVM이 메모리 할당받는게 1~2GB 정도였습니다. 그러면 Old 영역은 대략 1GB 내외의 영역이 있었습니다. Old 영역에 대한 GC를 돌때는 STW 가 많이 발생하는데, 이 **STW시간은 Old 영역의 크기에 비례**합니다. 왜냐하면 전체 메소드를 지우는 작업은 전체 메모리를 대상으로 해야 하기 때문에 전체 메모리를 훑어야 합니다. 그래서 현대로 오면서 메모리가 점점 더 커지고, 할당이 많이 되니까 JVM이 할당받는 메모리가 32GB면 Old 영역의 크기가 16GB 내외로 매우 커졌습니다. 이렇게 되면 full GC의 STW 시간이 똑같이 메모리 크기에 비례해서 커지게 됩니다. 이렇게 되면 GC가 한번 돌면 2~3초까지도 어플리케이션이 멈추게 되고, 그 동안 들어오는 요청 때문에 서비스 장애가 발생하는 문제가 발생했습니다. 

따라서 메모리를 물리적으로 나누지 않고 **논리적**으로 나누는 **G1** 이라는 방법이 나왔습니다. 물리적으로 영역을 나누어 놓았을 때는 Young 영역에서 Old 영역으로 객체를 옮길 때 메모리 복사를 해주어야 했지만, G1 에서는 논리적으로 나누어 놓았기에 flag 만 바꾸면 되고, 물리적으로 메모리 복사를 안해주어도 된다는 장점도 있습니다. 

### G1 Collector

**G1 GC**는 바둑판의 각 영역에 객체를 할당하고 GC를 실행합니다. 그러다가, 해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행합니다. 

<img width="533" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/96466023-f2b7-4785-bd0e-57c593be81ef">

G1 에서는 Young 영역과 Old 영역이 물리적으로 나뉘어 있지 않고, 각 구역의 크기는 모두 동일합니다. 바둑판 모양의 구역이 각각 eden, survivor, old 영역의 역할을 변경해 가면서 하고, humongous 라는 영역도 포함됩니다. 각 구역의 사이즈는 1MB에서 32MB 사이로 할당됩니다. 그리고 총 구역의 개수는 대략 2048개 입니다. 예를 들어, JVM에게 8GB의 메모리를 할당하면, JVM은 3~4MB 사이즈의 블럭을 대략 2000개 정도 생성할 것입니다. 

G1은 크게 **Young-Only Phase** 와 **Space Reclamation** 단계로 나누어집니다. 

G1 이 **Young GC** 를 하는 방법을 봅시다. 
1. 몇개의 구역을 선정하여 Young 영역으로 지정합니다. 
2. 이 Linear 하지 않는 구역에 객체가 생성되며 데이터가 쌓입니다.
3. Young 영역으로 할당된 구역에 데이터가 꽉 차면, GC를 수행합니다.
4. GC를 수행하면서 살아있는 객체들이 있으면, 그 구역을 Survivor 구역으로 지정합니다.  

몇번의 aging 작업을 통해서, 살아남은 객체들이 있다면 Old 구역으로 지정됩니다. 

G1의 **Old GC**는 CMS GC 의 방식과 비슷하며, 6가지 단계로 나뉩니다. 

1. **Initial Mark(STW)** : Old 영역에 있는 객체에서 Survivor 영역의 객체를 참조하고 있는 객체들을 표시합니다.
2. **Root region scanning** : Old 영역 참조를 위해 Survivor 영역을 훑습니다. 이 작업은 Young GC가 발생하기 전에 수행됩니다.
3. **Concurrent Mark** : 전체 힙 영역에 살아있는 객체를 찾습니다. 이때 Young GC가 발생하면 잠시 멈춥니다. 
4. **Remark(STW)** : 힙에 살아있는 객체들의 표시 작업을 완료합니다. 이때 snapshot-at-the-beginnning(SATB) 라는 알고리즘을 사용하며, 이것은 CMS GC에서 사용하는 방식보다 빠릅니다.  
5. **Cleaning (STW)** : 살아있는 객체와 비어 있는 구역을 식별하고, 필요 없는 객체들을 지웁니다. 그리고 비어 있는 구역을 초기화합니다.
6. **Copy (STW)** : 살아있는 객체들을 비어 있는 구역으로 모읍니다. 

G1 GC 는 GC를 가동하는 시간을 얼마나 설정한 것인가로 튜닝합니다. 따라서 시간을 설정해서 STW의 시간을 최소화할 수 있습니다. 

하지만 주의해야 할 점은, G1의 region 크기보다 더 큰 객체가 생성될 때는 알고리즘 대로 동작이 불가합니다. 이러한 큰 크기의 객체를 처리하기 위해 humongous 가 있는데, 일반적인 구역을 처리하는 알고리즘보다 비효율적이므로 구역 크기보다 큰 크기의 객체를 만들지 않도록 주의해야 합니다. 

## 파이썬에서 GC

CPython에서는 기본적으로 **reference counting** 방법을 사용하고 있습니다. 

reference count 은 아래와 같은 순서로 실행됩니다.

1. 객체 생성 : 객체 생성 시, 참조 카운트는 1로 설정됩니다.
2. 참조 추가 : 객체에 대한 새로운 참조 추가시 카운트가 1씩 증가합니다. (e.g. 다른 변수에 할당)
3. 참조 해제 : 참조가 더 이상 필요하지 않을 때, 참조 카운트가 감소합니다. (e.g. 변수가 범위를 벗어나거나 다른 객체로 대체되는 경우)
4. 객체 소멸 : 참조 카운트가 0이 되면, 해당 객체를 메모리에서 자동으로 해제합니다. 


{{< codeeditor >}}
import sys

a = [1, 2, 3]
b = a
print("a ref count : ", sys.getrefcount(a))

b = None
print("a ref count : ", sys.getrefcount(a))

c = 1
print("c ref count : ", sys.getrefcount(c))
{{< /codeeditor >}}

위 코드를 실행해보면, 첫번째는 3이 나옵니다. 이유는 `getrefcount()` 함수 자체가 a에 대한 임시 참조를 생성하기 때문입니다. 따라서 a객체 생성, b객체가 a객체 참고, `getrefcount()`가 다시 참고해서 3이 됩니다. b 객채를 None 으로 바꿔서 a객체 참조를 해제한 후에는 a에 대한 참조 횟수가 하나 감소합니다. 

마지막으로 객체 c에 1을 할당하고 참조횟수를 출력하면 예상치못하게 큰 값(저는 113인데 달라질 수 있습니다.) 이것은 파이썬의 **내부 최적화** 때문입니다. 파이썬에서 작은 정수들은 사전에 미리 할당되고, 전역적으로 재사용됩니다. 따라서 c = 1 과 같이 정수 객체를 할당하면, 새로 객체를 생성하는 것이 아닌 이미 존재하는 객체에 대한 참조를 얻게 됩니다.


### Reference cycle 문제

하지만 reference counting 만 사용하면 **(reference cycle)순환 참조 문제**가 생길 수 있습니다. 

```python
>>> container = []
>>> container.append(container)
>>> sys.getrefcount(container)
3
>>> del container
```

위 코드에서, `container` 는 자기 자신에 대한 참조를 합니다. 따라서 container 에 대한 참조를 삭제해도 여전히 내부적으로 자기 자신을 참조하고 있기 때문에 참조 카운트는 영원히 0이 될 수 없습니다. 따라서 reference counting으로 절대 정리될 수 없습니다. 따라서 객체들이 unreachable 상태가 되었을 때 순환 참조 문제를 해결하기 위한 추가적인 방법이 필요합니다. 이것은 cyclic garbage collector(GC) 가 합니다. 

3.13 버전부터 CPyton 은 2가지 형태의 GC가 있습니다. 

- 내장된 **기본 구현**은 thread-safety 를 위해 [GIL](https://docs.python.org/3/glossary.html#term-global-interpreter-lock)에 의존합니다.
- **free-threaded 구현**는 GC를 수행하는 도중 thread-safety 를 위해 다른 실행 중인 쓰레드들을 중지시킵니다.

2가지 GC는 같은 알고리즘을 사용하지만, 다른 자료구조에 대해 실행됩니다. 

### 메모리 구조와 객체 구조

GC는 파이썬 객체에 GC를 수행하기 위한 추가적인 필드들을 요구합니다. 이 필드들이 기본GC와 free-threaded GC에서 다릅니다.  

#### 기본 GC 

파이썬 객체의 C 구조는 아래와 같이 생겼습니다. 

```text
object -----> +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ \
              |                    ob_refcnt                  | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ | PyObject_HEAD
              |                    *ob_type                   | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ /
              |                      ...                      |
```

GC를 지원하기 위해, 메모리 구조는 아래와 같이 수정됩니다.

```text
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ \
              |                    *_gc_next                  | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ | PyGC_Head
              |                    *_gc_prev                  | |
object -----> +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ /
              |                    ob_refcnt                  | \
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ | PyObject_HEAD
              |                    *ob_type                   | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ /
              |                      ...                      |
```

이러한 방식으로 객체는 GC에 관한 정보가 필요할 때, 원래의 객체에 간단한 typecast 를 적용해 접근할 수 있습니다 : `((PyGC_Head *)(the_object)-1)`. 이 2가지 추가적인 필드들은 GC가 추적하는 객체들에 doubly linked list 를 유지하게 합니다. doubly linked list는 자주 사용되는 연산들을 굉장히 효율적으로 할 수 있게 해줍니다. GC가 관리하는 객체들은 여러 개의 집합으로 구분되고, 각자 doubly linked list를 가지고 있습니다. 이 여러개의 집합은 "세대"의 개념으로 나누어지는데, 기준은 얼마나 많이 GC 를 살아남았는지 입니다. 수거 과정에서 세대 안의 객체들은 reachable / unreachable 로 구분됩니다. doubly linked list는 하나의 객체를 하나의 파티션에서 다른 파티션으로 옮기기, 새로운 객체 추가하기, 객체 제거하기, 파티션 합치기를 포인터 업데이터를 통해 상수 시간안에 할 수 있게 해줍니다.

#### free-threaded GC

free-threaded GC에서, 파이썬 객체는 1-byte 의 `ob_gc_bits` 필드가 있습니다. 이 필드는 GC가 추적하는 객체를 식별하기 위해 사용되고, 객체 당 GC가 한번씩만 작동하고 GC과정 동안 reachable / unreachable 객체를 구분하기 위해 사용됩니다.


```text
object -----> +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ \
              |                     ob_tid                    | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ |
              | pad | ob_mutex | ob_gc_bits |  ob_ref_local   | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ | PyObject_HEAD
              |                  ob_ref_shared                | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ |
              |                    *ob_type                   | |
              +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+ /
              |                      ...                      |
```

### 순환참조 알아내기 

CPython 이 순환참조를 알아내기 위해 사용하는 알고리즘은 `gc` 모듈에 있습니다. GC는 컨테이너 객체(다른 객체에 대한 참조를 포함할 수 있는 객체)를 수거하는 것에만 집중합니다. 컨테이너 객체는 배열, 딕셔너리, 리스트, 클래스, 예외 모듈의 클래스 등이 있습니다. 순환참조가 발생하는 예시는 아래와 같습니다.

- 예외는 예외 자체를 포함하는 traceback 객체를 포함할 수 있습니다.
- 모듈-레벨 함수는 모듈의 딕셔너리를 참조하고, 이 모듈의 딕셔너리는 다시 모듈-레벨 함수의 엔트리들을 포함할 수 있습니다. 
- 클래스를 참조하고 있는 인스턴스에서, 클래스는 모듈을 참조하고 모듈은 모듈 안에 들어있는 모든 객체를 참조하기 때문에 순환 참조가 발생할 수 있습니다.
- 그래프와 같은 자료구조를 나타낼 때, 자기 자신을 가리키는 내부 링크를 가질 수 있습니다. 

```python
>>> import gc

>>> class Link:
...    def __init__(self, next_link=None):
...        self.next_link = next_link

>>> link_3 = Link()
>>> link_2 = Link(link_3)
>>> link_1 = Link(link_2)
>>> link_3.next_link = link_1
>>> A = link_1
>>> del link_1, link_2, link_3

>>> link_4 = Link()
>>> link_4.next_link = link_4
>>> del link_4

# Collect the unreachable Link object (and its .__dict__ dict).
>>> gc.collect()
2
```

GC는 스캔하고 싶은 후보 객체들로 시작합니다. 기본 GC에서, "스캔할 객체"들은 모두 컨테이너 객체 또는 그 일부일 수 있습니다. free-threaded GC에서는 컬렉터가 언제나 모든 컨테이너 객체들을 스캔합니다. 

목표는 **모든 unreachable 객체들을 식별**하는 것입니다. 컬렉터는 우선 reachable 객체들을 식별하는데, 그러면 자동적으로 남는 객체들은 unreachable 객체가 됩니다. 첫번째 단계는 후보 객체들 집합 바깥에서 **바로** 접근할 수 있는 객체들을 찾는 것입니다. GC를 지원하는 객체들은 알고리즘이 시작할 때 모두 reference count 로 초기화되는 추가적인 필드(`gc_ref`)를 가지고 있습니다. 따라서 알고리즘이 이 추가 필드를 이용해서 reference count를 변경하는 연산을 수행할 수 있습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b3380fde-668c-47f8-9c2d-fac5363bb15a)

그 다음 GC는 첫번째 리스트에 있는 모든 컨테이너를 순화하고, 컨테이너 내의 객체가 참조하고 있는 다른 객체의 `gc_ref` 값을 하나씩 감소시킵니다. 모든 객체들을 스캔한 이후에는, 후보 객체 바깥에서 참조되는 객체만 `gc_ref > 0` 입니다. (아래의 경우 `var1`에 의해 참조되는 `link1`)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6216de62-1a5f-4f34-bcec-65ee8b6a990f)

이때 `gc_ref == 0` 이라는 것이 unreachable 하다는 의미는 아닙니다. 진짜로 unreachable한 객체들을 찾기 위해서, GC는 컨테이너 객체들을 다시 스캔합니다. 이번에는 `gc_ref == 0` 인 객체들을 "잠정적으로 unreachable" 하다고 마킹하고, "잠정적으로 unreachable" 한 리스트로 옮깁니다. 아래 사진은 GC가 `link_3` , `link_4` 는 처리했고, 아직 `link_1` , `link_2` 는 처리하지 않은 상태입니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0ed62a64-13ce-474c-bb06-1a2c117e0acd)

그 다음에 GC는 다음 `link_1` 객체를 스캔합니다. 이 객체는 `gc_ref == 1` 이기 때문에, GC는 이것이 이미 reachable 이라는 것을 알고 아무것도 하지 않습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/50e4116c-38ec-4453-b0ae-ee8533580c32)

GC가 reachable 한 객체(`gc_ref > 0`)를 마주치면, 그것이 참조하고 있는 모든 객체들을 탐색하고 reachable 리스트의 끝으로 옮기고, 그 객체들의 `gc_ref` 필드를 1로 설정합니다. 아래 사진에서, `link_2` , `link_3` 은 `link_1`에서 접근할 수 있으므로 reachable 리스트로 옮겨진 결과입니다. 이 과정에서 `link_3` 이 reachable 하다는 것을 알게 되고, `gc_ref` 필드를 1로 설정해서 GC가 다시 방문했을 때 이 객체가 reachable 이라는 것을 알게 됩니다. 따라서 GC는 같은 객체에 대해 다시 동작하지 않습니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cfdb9b1f-f2ac-4e1e-bea5-a85d46121ece)

이 과정에서 "잠정적으로 unreachable" 이라고 판단된 `link_3`은 나중에 다시 reachable 리스트로 옮겨졌고 GC가 다시 방문했습니다. 이 과정은 객체 그래프 상의 BFS라고 할 수 있습니다. 모든 객체가 스캔된 이휴에, GC는 "잠정적으로 unreachable" 리스트에 있는 객체들은 실제로 unreachable하다고 판단하고, 가비지 컬렉팅을 수행합니다. 

#### 왜 unreachable 객체를 옮길까? 

예를 들어, 객체 A,B,C 를 이 순서대로 생성한다고 합시다. 이 객체들은 young generation 에 이 순서대로 나타납니다. 그 후 B->A, C->B 로 참조하고, 외부 변수가 C를 참조한다고 하면, 첫번째 GC 알고리즘 단계 이후에 refcount 는 A=0, B=0, C=1 일 것입니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3300aa5b-a329-4acc-bc70-ae60f8f26b26)


 알고리즘의 두번째 단계에서 A와 B를 찾으면, `gc_ref == 0` 이므로 unreachable 리스트로 옮겨집니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4af4a40e-f314-4816-aa2a-62c3057312ba)
 
그 후 C에서 접근 가능한 모든 객체들을 찾는 단계에서 B가 다시 reachable 리스트로 옮겨지고 A도 그 다음 옮겨집니다. 이때 `gc_ref`는 1로 설정됩니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cfe4da6e-47ba-487a-ad04-0db689f6c975)


따라서 A,B는 원래 reachable 리스트에 있다가 unreachable 리스트로 옮겨지고, 다시 reachable 리스트로 옮겨집니다. 왜 이렇게 디자인했을까요? 왜냐면 이 과정을 거치면 원래 생성된 순서인 A,B,C가 아니라 C->B->A 순서가 되기 때문입니다. 따라서 다음 모든 스캔들에서, A,B 객체는 옮겨지지 않습니다. 다음 GC가 시작될 때도 `var A`로부터 다시 도달 가능한 객체를 찾습니다. 이 시점에서 객체들은 이미 최적화된 상태로 배치되어 있습니다 (C -> B -> A 순서). 첫 번째 컬렉션에서 객체들이 도달 가능 상태로 재배치되었으므로, 참조 카운트가 정확히 설정되어 있습니다. 외부 변수 Var A로부터 C, B, A로 순차적으로 도달 가능하므로, 참조 관계가 변하지 않는 한 더 이상 객체를 이동할 필요가 없습니다. 따라서 초기 비용만 높고, 그 이후에는 비용을 절약할 수 있습니다. 

### 세대(generation)를 이용한 최적화

각 가비지 컬렉션에 걸리는 시간을 제한하기 위해, 기본 빌드의 GC(가비지 컬렉터) 구현은 인기 있는 최적화 기법인 **세대(generations)** 를 사용합니다. 이 개념의 주요 아이디어는 **대부분의 객체가 매우 짧은 수명을 가지며 생성된 후 곧 수집될 수 있다는 가정**입니다. 이는 많은 Python 프로그램에서 사실에 가깝다는 것이 입증되었습니다. 많은 임시 객체들이 매우 빨리 생성되고 파괴되기 때문입니다.

이 사실을 활용하기 위해, 모든 컨테이너 객체는 세 개의 공간/세대로 분리됩니다. 모든 새로운 객체는 첫 번째 세대(세대 0)에서 시작합니다. 이전 알고리즘은 특정 세대의 객체들만 대상으로 실행되며, 객체가 자신의 세대에서 컬렉션을 통과하면 다음 세대(세대 1)로 이동합니다. 여기서는 덜 자주 수집됩니다. 동일한 객체가 새로운 세대(세대 1)에서도 또 한 번 GC 라운드를 통과하면, 가장 마지막 세대(세대 2)로 이동하며, 여기서는 가장 덜 자주 수집됩니다.

GC가 실행 시점을 결정하기 위해, 컬렉터는 마지막 컬렉션 이후의 객체 할당 및 할당 해제 횟수를 추적합니다. 할당 횟수에서 할당 해제 횟수를 뺀 값이 `threshold_0`을 초과하면 컬렉션이 시작됩니다. 초기에는 세대 0만 검사합니다. 세대 0이 세대 1이 검사된 이후 `threshold_1`번 이상 검사되었다면, 세대 1도 검사합니다. 세대 2의 경우, 상황이 조금 더 복잡합니다. 이러한 임계값은 `gc.get_threshold()` 함수를 사용하여 확인할 수 있습니다.

```python
>>> import gc
>>> gc.get_threshold()
(700, 10, 10)
```


### 언어마다 다른 GC 전략을 사용하는 이유는 무엇일까?

왜 언어마다 다른 GC 방법을 사용할까요? 생각해보면, 언어의 특성과 밀접한 관련이 있습니다. 파이썬은 GIL 때문에 기본적으로 **싱글 쓰레드**로 동작합니다. reference counting 방식의 가장 큰 단점은 멀티쓰레환경에서 정확한 reference count를 측정하기 위해 복잡한 동기화 로직과 이 때문에 발생하는 성능 저하 문제입니다. 하지만 파이썬의 경우 싱글 쓰레드 기반이기 때문에 해당 단점이 문제가 되지 않고, 즉각적인 reference counting을 통한 별도 GC타임없이 GC를 수행할수있는 장점만 가져갈 수 있기 때문에 reference counting 방식을 채택한것 으로 판단됩니다.



## Reference
- [Garbage collection (computer science)](https://en.wikipedia.org/wiki/Garbage_collection_%28computer_science%29)
- [Tracing garbage collection](https://en.wikipedia.org/wiki/Tracing_garbage_collection)
- [Naver D2 - Java Garbage Collection](https://d2.naver.com/helloworld/1329)
- [Stackoverflow - Why Is Garbage Collection So Important?](https://stackoverflow.com/questions/1695131/why-is-garbage-collection-so-important)
- [Garbage-First (G1) Garbage Collector](https://docs.oracle.com/en/java/javase/17/gctuning/garbage-first-g1-garbage-collector1.html)
- 자바 성능 튜닝 이야기 ch.17, 이상민 저
- [Naver D2 - ZGC의 기본 개념 이해하기](https://d2.naver.com/helloworld/0128759)
- [IBM Developer - Inside memory management](https://developer.ibm.com/tutorials/l-memory/?mhsrc=ibmsearch_a&mhq=garbage%20collector)
- [Garbage Collection in Python](https://www.geeksforgeeks.org/garbage-collection-python/)
- [Python Developer's Guide - Garbage collector design](https://devguide.python.org/internals/garbage-collector/index.html)


