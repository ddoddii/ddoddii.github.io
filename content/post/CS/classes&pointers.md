+++
author = "Soeun"
title = "Class & Pointer"
date = "2023-04-03"
description = "파이썬에서 Class 와 Pointer"
categories = [
    "CS"
]
tags = [
    "자료구조"
]
image = ""
+++
## Class & Pointers

### Class

- Class : 쿠키를 굽는 틀이다. 내부에 쿠키를 정의하는 여러 가지 함수(method)를 정의한다.
- Instance : 쿠키틀 사용해서 실제로 만들어낸 쿠키들이다.

```python
class Cookie:
    def __init__(self,color):
        self.color = color
    def get_color(self):
        return self.color
    def set_color(self,color):
        self.color = color


cookie_one = Cookie('green') # 초록 쿠키 instance 생성 
cookie_two = Cookie('blue') # 파란 쿠키 instance 생성

print(cookie_one.get_color()) # green 출력
cookie_one.set_color('red')
print(cookie_one.get_color()) # red 출력
```

### Pointer

```python
num1 = 11
num2 = num1

print(id(num1)) #140727958956656
print(id(num2)) #140727958956656
```

num2 = num1 이라고 하면 둘의 id 는 똑같다 = point to the same address

```python
num1 = 11
num2 = num1
num2 = 22

print(id(num1)) #140710779218544
print(id(num2)) #140710779218896
```
num2 = 22 로 업데이트 한 후 둘의 id 를 출력해보면, id 가 달라진 것을 볼 수 있다.

```python
dict1 = {'value':11}
dict2 = dict1 
```
여기서 dict1, dict2 는 같은 id 값을 가진다 = 메모리 상에서 같은 곳을 point 한다.
여기서 dict2['value'] = 22 로 업데이트 하면 어떻게 될까?

```python
dict1 = {'value':11}
dict2 = dict1 
dict2['value'] = 22
print(id(dict1)) #4349953664
print(id(dict2)) #4349953664
```

id 가 달라졌던 위의 int 를 다룰 때와 다르게 두 개의 dictionary id 값이 여전히 같다.

왜냐면 integer 는 immutable 하고 변경될 수 없다. 하지만 dictionary 는 mutable 하고 변경될 수 있다. 

이 컨셉은 중요한데, Linked List 에서 dictionary 를 사용하면, Node 의 값이 바뀌어도 변수는 여전히 같은 Node로 point 한다. 

```python
dict1 = {'value':11}
dict2 = dict1 
print(id(dict1)) #4302620288
print(id(dict2)) #4302620288
dict3 = {'value':33}
dict2 = dict3
print(id(dict1)) #4302620288
print(id(dict2)) #4302620352
print(id(dict3)) #4302620352
```
이렇게 하면 처음에 dict1, dict2 의 id 값이 같고, dict2=dict3 를 해준 뒤에는 dict2 와 dict3 의 id 값이 같아진다. Linked List 에서 head 를 바꾸고 싶을 때 단순히 head= 다른 것 으로 해주면 된다 ! 

여기서 dict1 = dict2 까지 해주면 어떻게 될까 ? 그러면 {'value':11} 를 가리키는 변수가 더 이상 없고, 이 값에 접근할 수 없어진다. 그러면 python 에서는 Garbage Collection 이라는 기능을 통해 이 값을 없애버린다.

### Reference
Udemy, Data Structures & Algorithms sec.3