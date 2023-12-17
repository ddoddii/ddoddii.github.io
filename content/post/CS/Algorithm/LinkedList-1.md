+++
author = "Soeun"
title = "Linked List 개념 & 시간복잡도"
date = "2023-04-07"
summary = "Linked List - 1"
categories = [
    "CS"
]
tags = [
    "자료구조"
]
image = ""
+++

## Linked Lists

- **Linked List** 
  - Linked List 는 node 라는 것이 각 원소마다 할당 된다. 
  - 각 node 는 각 원소에 대한 정보와, 이웃 node 에 대한 정보를 가지고 있다.
  - Node 는 value,next 정보를 다 가지고 있다.
  - Linked List 는 이러한 정보를 다 어떻게 가지고 있는걸까? -> nested dictionary 구조를 생각하면 쉽다
  - 아래는 11 -> 3 -> 4 -> 7 로 연결된 Linked List 를 간단하게 dictionary 로 구현한 것이다. head = 11, head.next.next.value = 4 가 된다.
    ```python
    head = {
        "value" : 11,
        "next" : {
            "value" : 3,
            "next" : {
                "value" : 4,
                "next" : {
                    "value" : 7,
                    "next" : None
                }
            }
        }
    }
    print(head['next']['next']['value']) # 4
    ```

- **Singly Linked List**
  - collection of nodes that collectively form a linear sequence 
  - Linked List 는 사전에 정해진 크기의 사이즈를 가지지 않고, 요소의 개수만큼 space 를 차지한다.
  - 
    ![스크린샷 2023-08-03 오후 5 14 27](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/565c27b2-81e9-4618-a240-a2eba14ad065)


1. **Singly Linked List 의 마지막에 요소 추가하기**
   - 마지막 노드가 새로운 요소를 가리킨다.
   - tail 을 새로운 요소로 업데이트한다.
   - Linked List 의 사이즈를 +1 한다.
   - 노드의 개수에 상관없으므로 O(1) 의 시간복잡도
    ```python
    def add_last(L,e):
        newest = Node(e)
        newest.next = None
        L.tail.next = newest
        L.tail = newest
        L.size += 1
    ```

2. **Singly Linked List 의 마지막에 요소 제거하기**
    - 마지막 node 를 제거하기 위해서는 마지막 이전 node 에 접근해야 한다.
    - 마지막 이전 node 의 next 가 tail 이라는 정보를 없애줘야 한다.
    - 하지만 마지막 이전 node 에 접근하기 위해서는 head에서 시작하여 많은 traverse 를 해야 한다.
    - 시간복잡도 O(n) 

3. **Singly Linked List 의 처음에 요소 추가하기**
    - 새로운 요소의 next 가 head 이다.
    - L의 사이즈를 +1 한다.
    - node 의 개수에 상관 없으므로 O(1)
    ```python
    def add_first(L,e):
        newest = Node(e)
        newest.next = L.head
        L.head = newest
        L.size += 1
    ```
4. **Singly Linked List 의 처음 요소 제거하기**
    - head 를 head.next 로 옮긴다
    - node 의 개수에 상관 없으므로 O(1)
    ```python
    def remove_first(L):
        if L.head is None:
            return Error
        else:
            L.head = L.head.next
            L.size -= 1
    ```

5. **Singly Linked List 의 중간에 요소 추가하기**
    - 만약 3번째 자리에 새로운 요소 추가한다고 해보자. 
    - 추가하고자 하는 위치 전 node(2번째) 까지 traverse
    - 추가하고자 하는 새로운 요소가 원래 3번째 node 를 가리키도록 한다.
    - 시간복잡도는 O(n)


6. **Singly Linked List 의 중간에 요소 제거하기**
    - 만약 3번째 자리에 요소를 제거한다고 해보자. 
    - 제거하고자 하는 위치 전 (2번째 node) 가 제거하고자 하는 위치 후 (4번째 node) 를 가리키도록 한다.
    - 시간복잡도는 O(n)

7. **Singly Linked List 에서 요소 찾기**
    - 찾고자 하는 요소에 도착할 때까지 계속 traverse 해야 한다
    - 시간복잡도는 O(n)

### Linked List vs List

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9a2c0497-9c26-4403-af0e-607831467ffd)


### Reference 
- 2023-1 데이터 사이언스를 위한 자료구조 및 알고리즘, 연세대학교 송경우 교수님
- Udemy, Python Data Structures & Algorithms