+++
author = "Soeun"
title = "Big-O"
date = "2023-04-01"
description = "Big-O 의 시간복잡도에 대해 알아보자"
categories = [
    "CS"
]
tags = [
    "자료구조"
]
image = ""
+++

### Time Complexity

Big(O) : Worst case 를 생각한다 

- **O(n)** : n개를 실행하는데 n개와 정확하게 비례한다. 
  ```python
    def print_items(n):
        for i in range(n):
            print(i)
    ```
    - n개를 실행하는 시간이 언제나 n 에 비례한다. 
    
- **O(n^2)**
    ```python
    def print_items(n):
        for i in range(n):
            for j in range(n):
                print(i,j)
    ```
    - n * n = n^2 번 실행되었다. 

- **O(1)** 
    ```python
    def add_items(n):
        return n + n
    ```
    - n에 상관없이 실행시간이 똑같다(=상수이다).

- **O(logn)**
  - list = [1,2,3,4,5,6,7,8] 가 있을 때, 1을 찾기 위해 list 를 계속 반으로 쪼개보자. 8개의 요소를 가진 리스트에서 1만 남기기 위해 계속 list 를 반으로 쪼개보자. 
  [1,2,3,4,5,6,7,8] -> [1,2,3,4] -> [1,2] -> [1]
  3번 쪼개면 된다. -> 2^3 = 8 -> log2(8) = 3
  logn 번의 연산이 필요하다 !

![img](https://github.com/ddoddii/skills-for-DS/assets/95014836/290ae712-4efa-46c6-82e9-b3580b7e8b65)

### 규칙 

-  상수는 무시한다. 아래 코드는 n+n 번 실행되었다. 그렇지만 상수는 버리고 O(n) 이라고 작성한다. 
    ```python
        def print_items(n):
            for i in range(n):
                print(i)
            for j in range(n):
                print(j)
    ```

- Non-Dominant 가 아닌 것은 버리고 Dominant 만 남긴다. 아래 코드는 n^2 + n 번 실행되었다. 그러면 더 지배적인 n^2 만 남기고 O(n^2) 이라 표기한다.
     ```python
        def print_items(n):
            for i in range(n):
                for j in range(n):
                    print(i,j)
            for k in range(n):
                print(k)
    ```

- 다른 두 변수가 있을 때는 합칠 수 없다. 첫번째 경우에는 O(n) + O(n) 이 아니다. 왜냐하면 두 변수 a,b 를 다루고 있기 때문이다. 따라서 이 경우에는 O(a) + O(b) 라고 표기해야 한다. 마찬가지로 두번째 경우는 O(n^2) 이 아니라 O(a*b) 라고 표기해야 한다.
     ```python
        def print_items(a,b):
            for i in range(a):
                print(i)
            for j in range(b):
                print(j)
    ```
    ```python
        def print_items(a,b):
            for i in range(a):
                for j in range(b):
                    print(i,j)
    ```