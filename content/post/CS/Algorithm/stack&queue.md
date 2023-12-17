+++
author = "Soeun"
title = "Stack & Queue"
date = "2023-04-11"
summary = "Stack 과 Queue 개념"
categories = [
    "CS"
]
tags = [
    "자료구조"
]
image = ""
+++


## Stack 

- Stacks : a collection of objects that are inserted and removed according to the **last-in , first-out (LIFO)** principle
  - Stack 에서 remove 할 때, 가장 최근에 insert 된 것을 우선적으로 제거 
  - 비유하자면 프링글스 통 (가장 마지막에 넣은 것을 가장 먼저 먹는다) 같은 구조이다. 
  - ex. 인터넷 브라우저에서 가장 최근에 방문한 웹사이트를 저장할 때 stack 구조로 저장한다. 우리가 뒤로가기 버튼을 누르면 이 stack 에서 pop 한다. = 가장 최근에 방문한 웹사이트로 돌아간다. 

- Stack Methods
  - S.push(e) : e 를 stack 의 top 에 추가함
  - S.pop() : stack 의 top element 를 제거하고, 그 값을 리턴함
  - S.top() : stack 의 top element 를 리턴한다
  - S.is_empty() : stack 이 비어있으면 True 를 리턴함
  - len(S) : stack 안에 있는 element의 개수를 리턴함

- Stack Implement Analysis
  1. **Python Array**으로 구현 (Array Stack)

        ![img](https://github.com/ddoddii/Study-repo/assets/95014836/d7fbf526-bfce-42c1-bc63-67da9f207a39)

    - S.push(e) 랑 S.pop() 은 amortized O(1) 이다. 왜냐하면, push 와 pop 을 할때 stack 의 크기를 조절하는 resize 가 필요한 경우가 생긴다. 크기가 정해진 Array 에서 그 크기 이상으로 append를 하면, capacity 를 두배로 늘려주는 doubling 이 발생하는데, 이것의 시간복잡도가 O(n)이다. 따라서 0번째, 1번째, 2번째 ..push 는 O(1)이어도 n번째 push 가 O(n)이면 전체 시간복잡도는 amortized O(1) 이 된다. 
    - Amortized analysis : average runtime per operation over a worst-case sequence of operations

  2. **Linked List** 로 구현
    - Stack 에서 top 이 Linked List 의 head 일까, tail 일까?
    - Stack 에서는 push 와 pop 이 모두 top 에서 일어나기 때문에, insert 와 removal 이 쉬운 head 를 top 으로 생각하자 !

        ![img](https://github.com/ddoddii/Study-repo/assets/95014836/28102228-e059-490b-b042-8129f66216b4)

## Queue

- Queue : a collection of objects that are inserted and removed according to the **first-in, first-out (FIFO)** principle
  - 우리가 줄을 설 때, 가장 처음 온 사람이 가장 먼저 입장하는 것과 같다

- Queue Methods
  - Q.enqueue(e) : queue 의 가장 뒤에 e를 추가함
  - Q.dequeue() : queue 의 가장 첫번째 요소를 제거하고, 그 값을 리턴함
  - Q.first() : queue의 가장 첫번째 요소를 리턴함
  - Q.is_empty() : queue가 비어있으면 True 를 리턴함
  - len(Q) : queue 안에 있는 element의 개수를 리턴함

- Queue Implement Analysis
  1.  **Python Array** 로 구현
  - Queue 에서 dequeue 는 pop(0) 과 같다. 하지만 이때 비효율은 없을까?
  - pop(0)을 하면 언제나 worst-case behavior 인 O(n)이다. 왜냐하면 뒤의 모든 index 를 바꿔주어야 하기 때문이다. 이것을 더 효율적으로 바꿀 수 있는 방법은 없을까?
  - Method 1
    - dequeue 를 하고, index 를 바꾸는 것이 아니라, dequeue 한 영역은 None으로 채워 놓으면, O(1)로도 가능하다. 
    - 하지만, 위 방식으로 구현하게 되면 dequeue 를 하더라도 사이즈가 줄어들지 않는다. 즉 계속 enqueue 한 횟수에 비례하여 사이즈가 늘어나게 된다. 
  - Method 2
    - Modulo 연산을 활용하여, circular view 로 구현해보자.
    - E->F->G->H->I 순서대로 enqueue 되었다고 하자. 이때 dequeue 했을 때도 똑같은 순서대로 나와야 한다. 
    - (현재 index) mod N 을 했을 때 나오는 값을 가져오도록 구현해보자.
      ```python
      from queue import Empty

      class ArrayQueue:
          DEFAULT_CAPACITY = 10
          
          def __init__(self):
              self._data = [None] * ArrayQueue.DEFAULT_CAPACITY
              self._size = 0
              self._front = 0
              
          def __len__(self):
              return self._size
          
          def is_empty(self):
              return self._size == 0
          
          def first(self):
              if self.is_empty():
                  raise Empty('Queue is empty')
              
          def _resize(self,cap):
              old = self._data
              self._data = [None]*cap
              walk = self._front
              for k in range(self._size):
                  self._data[k] = old[walk]
                  walk = (1+walk) % len(old)
              self._front = 0
          
          def dequeue(self):
              if self.is_empty():
                  raise Empty('Queue is empty')
              answer = self._data[self._front]
              self._data[self._front] = None
              self._front = (self._front + 1) % len(self._data)
              self._size -= 1
              return answer
          
          def enqueue(self,e):
              if self._size == len(self._data):
                  self._resize(2*len(self._data))
              avail = (self._front + self._size) % len(self._data)
              self._data[avail] = e
              self._size += 1
      ```
    - capacity 가 10 인 queue 를 생각해보자.
      - 그리고 현재 self._front = 5 라고 가정하고, 3개의 원소가 채워져 있다고 생각해보자 (Index 가 4 인 원소를 dequeue 하고, self._front 가 5로 업데이트 된 상황)
      ![img](https://github.com/ddoddii/Study-repo/assets/95014836/2b43d926-981f-4690-b311-650c40c06871)
      - _front 와 _size 의 합은 8이 된다. 즉 8번째 위치에 enqueue 가 된다. 
      - 계속 enqueue 하면, 그 다음은 9번째, 그 다음은 0번째 index 에 enqueue 한다.
    - _resize 의 역할
      - 만약 self._size 가 미리 지정한 DEFAULT_CAPACITY 크기의 self._data 와 같아졌다면, resize 를 해야한다. 이때 원래 index에 맞게 transfer 하는 것이 resize 함수이다. 
       ![img](https://github.com/ddoddii/Study-repo/assets/95014836/3408510e-a22a-4a77-a5f0-919d0bb1ffbd)
  - Big-O Analysis

    ![img](https://github.com/ddoddii/Study-repo/assets/95014836/5f7d288e-c214-4ade-a9e1-4285cb3c012d)
    - enqueue 할 때 resize 를 해야 할 때 시간복잡도가 O(n)이므로 전체 Running Time 은 amortized O(1)이다.
    - dequeuq 할 때도 마찬가지로 원소개수가 너무 적어지면 size 를 줄이는 resize 를 수행할수도 있다.

  2. **Linked List** 로 구현
    - Linked List 에서 head 에서 제거하는 것이 더 낫고, 요소를 더하려면 tail 에 저장하는 것이 낫다.
    - Queue 에서는 FIFO 로 tail 에 저장하고, head 에 있는 것을 가져오면 된다. (Dequeue 는 head 에서, Enqueue는 tail 에서)
    - Linked List 에서 head, tail 이었던 것을 head = first , tail = last 라고 생각하면 된다.


### Reference 
- 2023-1 데이터 사이언스를 위한 자료구조 및 알고리즘, 연세대학교 송경우 교수님
- Python Data Structures & Algorithms , Udemy
