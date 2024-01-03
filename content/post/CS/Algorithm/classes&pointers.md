+++
author = "Soeun"
title = "Class & Pointer"
date = "2023-04-03"
summary = "파이썬에서 Class 와 Pointer"
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
### Inheritance 
- **"A natural way to organize various structural components of a software package is in a hierachical fashion"**
- 아래는 수업시간에 정말 헷갈렸고 머리가 터졌던 예시이다. 

**오류코드 1.**
```python
class Square:
    def __init__(self,side):
        self.side = side
    def area(self):
        return self.side * self.side
class SquarePrism(Square):
    def __init__(self,side,height):
        self.side = side
        self.height = height
    def area(self):
        base_area = super().area() #Square 의 area 함수를 상속받음
        lateral_area = self.side * self.height
        returen 2 * base_area + 4 * lateral_area
class Cube(SquarePrism):
    def __init__(self,side):
        super().__init__(self,side)
    def area(self):
        return super().area() * 100

x = Cube(2)
print(x.area()) # Error
```
- 위의 코드에서 Cube의 init 함수 내의 `super().__init__(self, side)` 를 보자. 
  - super() 라고 했으니 SquarePrism 의 __init__ 함수를 가져온다. SquarePrism 의 __init__ 함수는 인자가 2개 (side, height) 필요하다. 
  - x = Cube(2) 라고 했을 때 SquarePrism 의 side = Cube(2) , height = 2 가 된다.
  - SquarePrism 에서 super().area() 를 할 때, Square 의 area method 내 self.side * self.side 는 Cube(2) * Cube(2) 가 된다.
  - 하지만 우리는 Instance (Cube) 끼리의 곱셈은 정의한 적 없다. 
  - 따라서 위 코드는 오류가 난다 ! 

**오류코드2.**

```python
class Square:
    def __init__(self,side):
        self.side = side
    def area(self):
        return self.side * self.side
class SquarePrism(Square):
    def __init__(self,side,height):
        self.side = side
        self.height = height
    def area(self):
        base_area = super().area() #Square 의 area 함수를 상속받음
        lateral_area = self.side * self.height
        returen 2 * base_area + 4 * lateral_area
class Cube(SquarePrism):
    def __init__(self,side):
        super().__init__(self,side)
        self.side = side
    def area(self):
        return super().area() * 100

x = Cube(2)
print(x.area()) #2400
```
- 위 코드는 정상적으로 작동하지만, 옳게 고친 방법은 아니다.
  - Cube 의 __init__ 내에 `self.side = side` 를 추가했다.
  - 앞과 동일하게, 여전히 SquarePrism 에서 side = Cube(2) , height = 2 
  - 하지만, `self.side = side` 를 통해서, side = 2 로 변경된다.
  - 따라서 본의 아니게, side 자리에는 side 가, height 자리에도 side 가 들어간다.

**정답코드**

```python
class Square:
    def __init__(self,side):
        self.side = side
    def area(self):
        return self.side * self.side
class SquarePrism(Square):
    def __init__(self,side,height):
        self.side = side
        self.height = height
    def area(self):
        base_area = super().area() #Square 의 area 함수를 상속받음
        lateral_area = self.side * self.height
        return 2 * base_area + 4 * lateral_area
class Cube(SquarePrism):
    def __init__(self,side):
        super().__init__(side=side,height=side)
    def area(self):
        return super().area() * 100

x = Cube(2)
print(x.area())
``` 
- 위의 코드에서 Cube의 __init__ 함수에 `super().__init__(side=side,height=side)` 를 보자
  - Cube 는 인자를 1개 받는다 (side)
  - Cube 는 정육면체이므로 side, height 가 같다. 따라서 height = side 를 해주면, height 에도 side 와 같은 값이 들어가서 Cube class 가 완성된다 !



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
- 2023-1 데이터 사이언스를 위한 자료구조 및 알고리즘, 연세대학교 송경우 교수님
- Udemy, Data Structures & Algorithms sec.3