+++
author = "Soeun"
title = "Python 코드로 알아본 Concurrency vs. Parallelism"
date = "2024-01-08"
summary = "실제 파이썬 코드로 비교한 동시성 vs 병렬성, 멀티쓰레딩 vs. 멀티프로세싱"
categories = [
    "CS"
]
tags = [
    "python",
    "concurrency"
]
slug = "python-concurrency"
+++

## Concurrency

Concurrency(동시성) 의 사전적 정의는, 2가지 이상의 것들이 동시에 일어나는 것이다. 병렬도 비슷한 것을 의미하기 때문에 이 정의가 그다지 많은 도움이 되지는 않는다.

CPU 는 오직 하나의 태스크만 처리할 수 있다. 만약 여러 개의 태스크가 주어지면, CPU 는 이 태스크 간에 스위칭한다. 스위칭을 굉장히 빠르게 하기 때문에 여러 개의 태스크를 "동시에" 하는 것처럼 보인다.  즉, 여러 개의 태스크가 병렬적으로 동시에 실행되는 것처럼 보이지만, 이것은 parallel(병렬)이 아니다. 이것은 concurrent(동시) 이다.  

![image](https://github.com/ddoddii/algorithm/assets/95014836/eb081eaa-305b-4079-a2ad-e11c5e5c102e)

![image](https://github.com/ddoddii/algorithm/assets/95014836/385ab33f-a32b-4dbf-bcd0-89d9f6eed5e7)


동시성은 넓은 의미에서 멀티스레딩으로 이해될 수 있다. 일반적으로 동시적 애플리케이션을 생성하는 많은 방법들이 있으며, 스레딩은 그 중 하나에 불과하다. 비동기 코드로도 동시성을 구현할 수 있다. 그러나 더 넓은 관점에서 보면, 둘 다 일시 중지하고 재개할 수 있는 일련의 명령들을 의미한다. 

Concurrent Computing(동시 컴퓨팅)이라는 프로그래밍 패러다임이 있다. 이는 한 번에 하나 이상의 프로세스가 동시에 수행되는 것처럼 보이도록 코드를 작성하는 것을 포함한다. **실제로는 동시에 실행되지 않지만**, 이것을 **동시 프로그래밍**이라고 한다. 

그렇다면 **thread(쓰레드)** 란 무엇일까?

스레드는 운영 체제가 서로 의존성 없이 처리하고 관리할 수 있는 가장 작은 작업 집합이다. 실제 구현은 다양한 운영 체제마다 다르다.

파이썬은 스레드 생성 및 관리를 위한 강력한 스레딩 모듈을 제공한다. 이 모듈을 사용하면 개발자는 파이썬 내에서 여러 스레드를 생성하고 제어할 수 있으며, 이를 통해 멀티태스킹 및 동시성 작업을 수행할 수 있습니다. 그러나 파이썬의 **글로벌 인터프리터 락(GIL)** 때문에, 순수 파이썬 코드에서 진정한 병렬 처리는 제한적이다. 따라서 I/O 바운드 작업과 같이 블로킹이 주된 문제인 경우에 파이썬 스레딩이 가장 유용하다. 

GIL 때문에 **파이썬은 한 시점에 하나의 스레드만 파이썬 코드를 실행할 수 있다**. 따라서, CPU 바운드 작업에서는 여러 스레드가 동시에 실행되어도 실제로는 하나의 스레드만 CPU 자원을 사용하고 나머지는 대기 상태에 있게 된다. 이로 인해 멀티코어 프로세서의 이점을 충분히 활용하지 못하게 된다.

반면, I/O 바운드 작업에서는 대부분의 시간이 데이터를 읽거나 쓰는 데 소비된다. 이러한 작업에서는 스레드가 데이터를 기다리는 동안 다른 스레드가 실행될 수 있어, GIL의 영향이 상대적으로 적다. 이런 이유로 I/O 바운드 작업에서 파이썬의 멀티스레딩은 여전히 유용하다.

---

{{< alert icon="comment" >}}
**글로벌 인터프리터 락(GIL)** 이 뭔가요 ?
{{< /alert >}}


여러 개의 스레드가 파이썬 바이트코드를 한번에 하나만 사용할 수 있게 락을 거는 것을 의미한다. 쉽게 말해서 하나의 스레드만 파이썬 인터프리터를 제어할 수 있도록 하는 뮤텍스라고 보면 된다.

이게 어떤 의미냐면, 파이썬 프로그램은 특정 시점에 오직 하나의 스레드만 실행된다는 것이다. 파이썬 멀티 스레드 프로그램에서 멀티 스레드가 싱글 스레드처럼 동작하는 성능병목 현상을 발견할 수 있다.

GIL 이 등장한 배경을 이해하려면, 파이썬의 메모리 관리 방식에 대해 알아야 한다. 파이썬은 **레퍼런스 카운팅**으로 메모리를 관리한다. 레퍼런스 카운팅은 파이썬에서 객체의 수명을 관리하는 주된 방법이다. 각 객체에는 **참조 카운트**라는 값이 있으며, 이 값은 객체에 대한 참조(레퍼런스)의 수를 나타낸다. 객체가 다른 변수나 객체에 의해 참조될 때마다 이 카운트가 증가하고, 참조가 해제될 때마다 감소합니다.

참조 카운트가 0이 되면, 즉 더 이상 해당 객체를 참조하는 것이 없을 때, 파이썬의 **가비지 컬렉터**는 그 객체를 메모리에서 제거한다. 

![image](https://github.com/ddoddii/algorithm/assets/95014836/2d0f40e0-bb69-4874-af5f-a8adb3a54997)



문제는 이 레퍼런스 카운팅 변수가 멀티 스레드 환경에서 두 스레드가 동시에 값을 늘리거나 줄이는 **Race Condition**이 발생할 수 있다는 것이다. 이러한 상황이 발생하면 메모리 누수가 발생하거나 객체에 대한 참조가 남아있는 데도 메모리를 잘못 해제할 수 있다.

따라서 GIL은 멀티 스레드 프로그램에서 이러한 레퍼런스 카운팅에 의해 발생할 수 있는 문제를 미리 예방하고자 한다.

----

{{< alert icon="comment" >}}
**파이썬이 CPU bound 연산에 제한적이라면, 고수준의 CPU 연산을 필요로 하는 머신러닝 / 딥러닝에서 메인 언어로 사용되는 이유는 무엇일까?**
{{< /alert >}}


사실, 파이썬의 딥러닝 / 머신러닝 라이브러리는 내부적으로 고성능 언어(C) 로 작성되어 있다. Numpy 는 C로 작성되어 고속 계산을 제공한다. 

또한, 딥러닝에서 중요한 GPU 는 CPU bound 문제를 완화시킨다. GPU 는 병렬 처리에 매우 효과적이며, 대부분 딥러닝 라이브러리는 GPU 를 활용하여 대규모 연산을 효율적으로 처리한다. 

CPU bound 에 대한 또 다른 접근법은 multiprocessing 을 사용하는 것이다. 멀티프로세싱을 사용하면,  별도의 프로세스에서 코드를 실행할 수 있으며, 이는 각 프로세스가 자체 메모리 공간을 가지므로 GIL의 제한을 우회할 수 있다. 

----

{{< alert icon="comment" >}}
**Multithreading vs. Multiprocessing** ?
{{< /alert >}}

**Multithreading** 

- 하나의 프로세스 내에서 여러 스레드를 생성하여 동시에 작업을 수행하는 기법
- **메모리 공유**: 멀티스레딩에서 모든 스레드는 부모 프로세스의 메모리 공간을 공유한다. 이는 데이터 공유가 용이하게 하지만, 동시성 관련 문제(예: Race condition)를 야기할 수 있다.
- **오버헤드**: 스레드 간의 컨텍스트 전환은 프로세스 간 전환보다 가볍고 빠르다. 따라서 멀티스레딩은 상대적으로 오버헤드가 적다.
- **GIL의 영향**: 파이썬에서는 GIL(Global Interpreter Lock)로 인해 한 번에 하나의 스레드만이 실행될 수 있어, 멀티코어 시스템에서의 병렬 CPU 작업에는 적합하지 않을 수 있다.

**Multiprocessing**

- 멀티프로세싱은 여러 개의 독립적인 프로세스를 생성하여 동시에 작업을 수행하는 기법
- **메모리 분리**: 각 프로세스는 독립적인 메모리 공간을 가진다. 이는 메모리 공유 문제를 방지하지만, 프로세스 간 데이터 공유는 복잡해질 수 있다.
- **오버헤드**: 프로세스 간 컨텍스트 전환은 스레드보다 무겁고 느리다. 하지만 각 프로세스가 독립적이기 때문에 멀티코어 환경에서 진정한 병렬 처리가 가능하다.
- **GIL 우회**: 파이썬의 멀티프로세싱은 각 프로세스가 별도의 GIL을 가지므로, 파이썬의 GIL 제약으로부터 벗어날 수 있다.

---

## 파이썬 코드 예시

wikipedia 에서 238개 나라의 html 정보를 크롤링하는 코드를 보자. 

```python
import time  
import requests  
from bs4 import BeautifulSoup  
from urllib.parse import urljoin  
  
  
def get_links():  
    countries_list = (  
        "https://en.wikipedia.org/wiki/List_of_countries_by_population_(United_Nations)"  
    )  
    all_links = []  
    response = requests.get(countries_list)  
    soup = BeautifulSoup(response.text, "lxml")  
    countries_el = soup.select("td .flagicon+ a")  
    for link_el in countries_el:  
        link = link_el.get("href")  
        link = urljoin(countries_list, link)  
        all_links.append(link)  
    return all_links  
  
  
def fetch(link):  
    response = requests.get(link)  
    with open(link.split("/")[-1] + ".html", "wb") as f:  
        f.write(response.content)  
  
  
if __name__ == "__main__":  
    links = get_links()  
    print(f"Total pages: {len(links)}")  
    start_time = time.time()  
    # This for loop will be optimized  
    for link in links:  
        fetch(link)  
  
    duration = time.time() - start_time  
    print(f"Downloaded {len(links)} links in {duration} seconds")
```

<img width="411" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/4c02ceec-af07-4bb7-a423-fc19426dc885">

무려 296초나 걸렸다.

이제 동시성을 이용해서 for loop 를 최적화 해보자. 쓰레드를 쉽게 만들고 실행할 수 있는 `ThreadPoolExecuter` 라이브러리를 사용할 것이다.

```python
from concurrent.futures import ThreadPoolExecutor

"""
나머지 코드는 동일
"""

if __name__ == "__main__":  
    links = get_links()  
    print(f"Total pages: {len(links)}")  
    start_time = time.time()  
    # optimized code multi-threading 
    with ThreadPoolExecutor(max_workers=16) as executor:  
        executor.map(fetch, links)  
  
    duration = time.time() - start_time  
    print(f"Downloaded {len(links)} links in {duration} seconds")
```


<img width="408" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/276ae03e-9e47-4534-bb39-d31dc78fdedd">

20초 밖에 안걸렸다 !! 👏

여기서 executer 는 `fetch()` 함수를 모든 link 들에게 적용하고 결과를 반환한다. 쓰레드의 최대 수는 `max_workers` 에 의해 제어된다. 

위의 코드는 **I/O 바운드** 작업이라서, 시간이 크게 단축되었다. 위의 코드는 여러 웹 페이지를 다운로드하는 작업을 수행하는데, 각 페이지를 다운로드하는 과정은 주로 네트워크 I/O에 의존한다.

`ThreadPoolExecutor`를 사용하면 여러 스레드가 동시에 다른 페이지를 다운로드할 수 있게 되므로, 전체 다운로드 시간을 크게 단축시킬 수 있다. 이는 각 스레드가 네트워크 요청을 기다리는 동안 다른 스레드가 실행될 수 있게 하여 I/O 바운드 작업에서 멀티스레딩의 이점을 활용한다.

I/O 작업에서 ThreadPoolExecuter 를 사용하면, 하나의 스레드가 데이터를 기다리는 동안 대기 상태가 되었을 때 다른 스레드가 CPU 를 사용할 수 있다. 즉, 한 스레드가 I/O 작업을 기다리는 동안 다른 스레드가 실행될 수 있다. 

## Parallelism

앞의 부분에서는, 우리는 단일 프로세서의 멀티 쓰레드에 대해 살펴보았다. 그러나 대부분의 프로세서는 하나 이상의 코어를 가지고 있다. 경우에 따라서는 한 기계에 여러 프로세서가 있을 수 있다.

병렬 컴퓨팅이 한 예시이다. 이는 여러 프로세서가 동시에 많은 프로세스를 수행하는 컴퓨팅 유형이다. 이러한 병렬 처리를 달성하기 위해서는 특별한 프로그래밍이 필요하다. 이것을 병렬 프로그래밍이라고 하며, 여러 CPU 코어를 활용하도록 코드를 작성한다. 이 경우에는 실제로 여러 프로세스가 병렬로 실행된다. 다음 이미지는 병렬성의 개념을 나타낸다.

![image](https://github.com/ddoddii/algorithm/assets/95014836/b67bb6c5-ace9-44f9-89c0-e93862b8e3e7)

![image](https://github.com/ddoddii/algorithm/assets/95014836/ddbdb032-b45c-4435-b793-4c11b9595479)


그러면 이제 병렬성을 활용해서 성능을 개선해보자. 파이썬에서 병렬성은 멀티태스킹을 통해 이룰 수 있다. 이것은 여러 프로세서를 사용해서 링크를 동시에 다운로드 할 수 있게 해준다. 

파이썬은 내 기계에 몇개의 프로세서가 있는지 알려주는 `cpu_count()` 함수를 제공한다. 

```python
print(f"My computer has {cpu_count()} CPUs")
```

<img width="208" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/6ba86bd5-3ad2-4f02-a384-cfa150d935d2">

내 컴퓨터는 11개의 CPU 가 있다. 

위의 예시를 활용해서, 멀티 프로세싱 방식으로 개선해보자.

```python
from multiprocessing import Pool, cpu_count

"""
나머지 코드는 동일
"""
if __name__ == "__main__":  
    links = get_links()  
    print(f"Total pages: {len(links)}")  
    start_time = time.time()  
    # optimized code with multi-processing
    with Pool(cpu_count()) as p:  
        p.map(fetch, links)  
  
    duration = time.time() - start_time  
    print(f"Downloaded {len(links)} links in {duration} seconds")

```

<img width="411" alt="image" src="https://github.com/ddoddii/algorithm/assets/95014836/43858bbb-99e2-419c-816c-93067248df8c">

총 29초가 걸렸다 ! 

이 방식은, `cpu_count()` 를 계산해서 사용 가능한 CPU 코어의 수만큼 프로세스(`Pool`)를 생성한다. `p.map(fetch, links)`는 `fetch` 함수를 `links`의 각 항목에 병렬적으로 적용한다. `map` 메서드는 주어진 함수를 입력 리스트의 각 요소에 적용하고, 각 프로세스에서 작업을 수행한다.

`with` 문을 사용하면 `Pool` 객체가 컨텍스트 매니저로 작동하여, 작업이 끝나면 자동으로 프로세스 풀을 종료하고 모든 리소스를 정리한다. 

멀티프로세싱을 사용함으로써, 프로그램은 여러 CPU 코어를 활용하여 병렬로 작업을 수행할 수 있다. 이는 특히 CPU를 많이 사용하는 작업에서 성능을 크게 향상시킬 수 있다.

## Concurrency vs. Parallelism

| Concurrency(동시성)                                  | Parallelism(병렬성)                                           |
| ---------------------------------------------------- | ------------------------------------------------------------- |
| 동시에 실행되는 것처럼 보이는 것                     | 실제로 동시에 실행되는 것                                     |
| 논리적 개념                                          | 물리적 개념                                                   |
| 싱글코어, 멀티코어에 가능                            | 멀티코어에서만 가능                                           |
| 한번에 많은 일을 다루는 것 = 여러 명령을 관리하는 것 | 한번에 많은 일을 처리하는 것 = 동시에 여러 명령을 실행하는 것 |
| 멀티쓰레딩, 비동기 프로그래밍을 사용                                        | 멀티태스킹을 사용                                               |
| 중단(interruptions)에 관한 것                                                     | 격리(isolation)에 관한 것                                                              |

## Concurrency + Parallelism

![image](https://github.com/ddoddii/algorithm/assets/95014836/7d458ab6-96bf-4fa4-95ce-ec45a6cd2f71)

여러 개의 CPU 가 여러 개의 쓰레드를 관리하고 있다면, 병렬성 + 동시성 둘 다 이다. 

## Summary

위 실험은, 멀티 프로세싱 / 멀티 쓰레딩 이 어디에 강점이 있는지 보여준다. 위의 코드는 대기 시간이 많은 I/O 바운드 작업이다. **멀티쓰레딩**은 I/O 바운드 작업에서 효율적이다. 한 스레드가 I/O 작업으로 인해 대기 상태일 때, 다른 스레드가 실행되어 자원을 효율적으로 사용할 수 있다. 파이썬의 GIL(Global Interpreter Lock)은 CPU 바운드 작업에서 멀티스레딩의 효율을 제한하지만, I/O 바운드 작업에서는 그 영향이 크지 않다.

반면, **멀티프로세싱** 은 독립적인 여러 프로세스를 생성하여 작업을 수행한다. 각 프로세스는 별도의 메모리 공간을 가지며, 이로 인한 오버헤드가 존재한다. 멀티프로세싱은 CPU 바운드 작업에 더 적합하다. 각 프로세스가 독립적으로 CPU를 사용할 수 있어, 멀티코어 환경에서 병렬 처리의 이점을 더 잘 활용할 수 있다. I/O 바운드 작업에서는 프로세스 간의 컨텍스트 전환 및 메모리 관리로 인한 추가적인 오버헤드가 오히려 성능을 저하시킬 수 있다.

위 코드의 경우, 멀티스레딩이 멀티프로세싱보다 더 빠른 결과를 보인 것은, I/O 바운드 작업의 특성상 CPU 사용보다는 I/O 대기 시간이 주된 병목 지점이기 때문이다. 멀티스레딩은 I/O 작업 중에 다른 스레드가 CPU를 활용할 수 있게 해주는 반면, 멀티프로세싱은 각 프로세스의 메모리 공간 관리와 컨텍스트 전환에 더 많은 시간을 소요하게 된다. 따라서 I/O 바운드 작업의 경우, 멀티스레딩이 멀티프로세싱에 비해 더 효율적인 선택일 수 있다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Concurrency vs. Asynchronous Programming**</span></span> 

동시성은 더 큰 개념이고, 비동기 코드는 동시성을 달성하기 위한 하나의 방법이다. 비동기 작업은 병렬로 수행될 수도 있고, 단순히 주 스레드가 다른 작업을 수행하도록 하면서 비동기 작업이 어떤 자원을 기다리는 동안 자유롭게 할 수도 있다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Concurrency vs. Multithreading**</span></span> 

동시성은 더 큰 개념이다. 멀티스레딩은 병렬로 여러 스레드를 실행하여 동시성을 달성하는 한 방법이다. 그러나 동시성이 항상 작업이 병렬로 실행된다는 것을 의미하지는 않는다; 그것은 작업들이 동시에 실행되는 것처럼 관리된다는 것을 의미합니다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Asynchronous Programming vs. Multithreading**</span></span> 

비동기 프로그래밍은 멀티스레딩 없이도 달성될 수 있다. 반대로, 멀티스레드 시스템은 동기적일 수 있으며, 각 스레드가 그 작업이 완료될 때까지 기다리고 나서야 다음으로 넘어간다. 그러나 많은 현대 시스템에서는 비동기성과 멀티스레딩이 종종 손에 손을 잡고 있으며, 비동기 작업은 병렬성을 달성하기 위해 다른 스레드로 오프로드된다.


## Reference
- https://oxylabs.io/blog/concurrency-vs-parallelism
- https://bloofer.net/114

