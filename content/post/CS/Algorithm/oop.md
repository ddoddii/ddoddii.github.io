+++
author = "Soeun"
title = "Object-Oriented Programming"
date = "2023-04-02"
summary = "OOP 의 특징"
categories = [
    "CS"
]
tags = [
    "객체지향"
]
image = ""
+++

## Object-Oriented Programming

### Procedure-Oriented vs. Object-Oriented

- Procedure-Oriented(절차지향)
  - 동작 위주 프로그래밍이다. 비유하자면 "행위" 위주이다. 
  - 어떤 사람이 오피스를 나와 음식점을 가는 과정을 보자.
  - 사람이 오피스를 **나간다** -> 엘레베이터를 타고 **내려간다** -> 음식점에 **입장한다** -> 음식을 **먹는다**
  - action, function 을 구현한다.
  
- Object-Oriented(객체지향)
  - 객체 위주이다. 비유하자면 "명사" 위주이다. 
  - 어떤 사람이 오피스를 나와 음식점을 가는 과정을 보자.
  - 사람이 **오피스**를 나간다 -> **엘레베이터**를 타고 내려간다 -> **음식점**에 입장한다 -> **음식**을 먹는다
  - object 와 actors 를 구현한다.
  - 행동은 object 들이 행하는 method 이다. 

### Object-Oriented Design Principles

- Modularity
  - 하나의 기능을 하는 여러 모듈들이 합쳐저서 복잡한 기능을 하는 소프트웨어 시스템을 구성한다.
- Abstraction
  - 가장 근본적인 부분까지 시스템을 쪼개야 한다.
  - abstract data type(ADTs)는 각 operation 이 어떤 것을 하는지 구체화한다. 하지만 어떻게(how) 하는지는 명시하지 않는다.
- Encapsulation 
  - 실제로 구현한 부분을 외부에 드러나지 않도록 해야 한다.
  - 프로그래머들은 외부에서 캡슐 속 함수들에 의존한다는 걱정은 하지 않고 컴포넌트를 구현할 자유가 있어야 한다.