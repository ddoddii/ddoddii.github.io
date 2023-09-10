+++
author = "Soeun"
title = "프로그래밍 언어 구조 Intro"
date = "2023-09-10"
description = "Programming Paradigm, Abstraction, Defining Language"
categories = [
    "CS"
]
tags = [
    "프로그래밍언어"
]
image = ""
+++

## Machine Language(기계 언어)
- 기계어는 컴퓨터가 직접 읽고 이해할 수 있는 비트 단위로 쓰여진 언어이다.
- 각 줄의 언어는 16bits 로 이루어져 있다. (이진수) 

    <img width="295" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dd24ab04-0058-409b-acc5-9f731cdf0791">
- 프로그램 실행은 코드의 첫번째 줄부터 시작된다.
  - Code is fetched from memory, decoded(interpreted), and excuted.
- Control 이 다음 줄로 넘어가고, halt instruction 이 있을 때까지 계속 반복한다. 

## Assembly Language
-  Assembly Language: 기계어와 일대일 대응이 되는 컴퓨터 프로그래밍의 Low-level 언어이다. 
-  Assembler : 어셈블리 언어를 기계가 알아들을 수 있는 binary code 로 변환해주는 프로그램
-  Loader : 기계어를 컴퓨터 메모리로 load 해주는 프로그램
-  어셈블리어는 이진수로 표현되는 기계어보다는 symbol 이 있다는 발전점이 있었지만 여전히 한계들이 존재했다.
   -  수학적 표기들에 대한 추상화가 부족
   -  주어진 컴퓨터 하드웨어에 의존성이 강하다(A컴퓨터에서 작성된 어셈블리 언어가 B컴퓨터에서 작동 안 할 수 있다)
-  어셈블리 언어는 임베디드 시스템, 커널 프로그래밍, 보안 등 Low-level 작업에 아직까지 쓰이고 있다.

## Computation without von Neumann Architecture
- Fortran, Algol 과 같은 High level 언어들은 여전히 폰 노이만 구조를 따르고 있었다.
  - Memory 영역에 데이터와 프로그램이 저장됨
  - 각각 다른 CPU 들이 메모리서부터 받은 명령들을 한 번에 명령어 하니씩 순차적으로 처리
    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0d162196-e04f-40a4-a32b-37084c1ad1df)
- 폰 노이만 구조의 한계
  - CPU에서 계산을 빠르게 처리할 수 있어도, 메모리에서 불러오는 속도가 느리다면 전체 시스템의 성능 저하가 일어난다(Von-Neumann Bottleneck)
- 해결방법
  - 언어들이 하드웨어 모델을 기반으로 하는 것이 아니라, 문제 해결 과정에 적합한 언어이어야 한다!ㅇ

## Abstractions
- 추상화의 2가지 측면
  - Hiding the details
  - Showing the important parts
- 프로그래밍 언어의 추상화의 2가지 종류
  - Data Abstraction : 데이터의 특성을 단순화함 (ex. number, strings, trees)
  - Contral Abstraction : 프로그램을 실행하는 흐름을 단순화함 (ex. loops, conditional statements, procedure calls)
- Abstraction Levels
  - Basic Abstraction : 가장 지역적인 기계 정보를 수집한 추상화
  - Structured Abstraction : 보다 전역적인 프로그램 구조에 대한 추상화
  - Unit Abstraction : 단위 프로그램 전체 정보에 대한 추상화 

### Data
- Basic Abstraction 
  - 컴퓨터 내부의 자료 표현을 숨김
  - Variables : 변수들이 어느 컴퓨터 메모리에 저장되어 있는지는 알 필요 X
  - Data types : 데이터 값들에 붙여진 이름
  - Declration : 어떠한 값에 이름과 데이터 구조를 주는 것
- Structured Abstraction 
  - Data Structure : 관련된 데이터 값들의 집합을 하나의 unit 으로 만든 것 
  - ex. record, array, file
- Unit Abstraction 
  - 자료의 생성과 사용에 대한 정보를 한 장소에 모아두고, 자료의 세부사항에 대한 접근을 제한
    - ex. Java의 class
  
### Control
- Basic Abstraction 
  - 기계 명령들을 모아 좀 더 사람이 이해하기 쉬운 방식으로 표기 
  - syntatic sugar : 복잡한 notation 을 좀 더 직관적이고 단순화해서 표기할 수 있는 것 (ex. x+=10)
- Structured Abstraction 
  - 프로그램에서 어떤 검사된 값에 따라 분할된 명령어의 한 그룹을 수행하도록 하는 것
  - ex. if 문 , for 문
- Unit Abstraction 
  - 라이브러리처럼 논리적으로 연관된 절차들의 집합을 추상화. 

## Programming Paradigms
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e2725c34-ba1e-4680-8313-94a9645ec03f)
- Imperative paradigm(명령형)
  - 명령형 프로그램은 컴퓨터가 수행할 명령들을 순서대로 써 놓은 것
  - 구체적으로 어떤 결과 값을 얻으려면 어떻게 해야하는지 자세하게 설명함
  - 폰 노이만 병목현상이 발생할 수 있다.

- Functional paradigm
  - Lambda calculus 의 functions를 기반으로 한다
- Logic paradigm
  - 선언 언어의 일종이다.ㅏ 
  - 논리 문장을 이용하여 프로그램을 표현하고 계산을 수행하는 개념이다. 
- Object-oriented paradigm
  - 명령형 패러다임의 문제점을 보완한다.
  - class 구조를 이용하여 코드를 재사용 할 수 있다. 


## Language Defining Methods
- syntax : structure of program
  - Tokens : 언어를 이루고 있는 어휘들
  - Lexical structure : 어휘 구조
  - context-free grammar

- semantics : meaning of program
  - Formal methods : operational, denotational and axiomatic semantics

## Language Translator
- 기계가 이해할 수 있도록 코드를 번역하는 것
- 두가지 종류
  - Interpreter : 프로그램 자체를 그냥 실행 (Line-by-line)
  - Compiler : 기계어로 변환을 먼저 한 후 실행
  - pseudo-interpreter : 위 두가지 방식의 hybrid 

## Error Handling
- Error 의 종류
  - Syntax errors
  - Semantic errors
    - Static semantic errors : compile 시간 동안 탐지 가능 (ex. int 변수에 float 할당)
    - Dynamic semantic errors : 실행 중에만 탐지 가능 (ex. 0으로 나누는 에러)
  - Logic errors : 개발자만 알 수 있는 로직 에러

이번 주는 앞으로 배울 내용의 개괄 느낌으로 정리했다. 이것이 기본 뼈대이고 앞으로 더 살을 붙여 나가며 공부해야겠다 ! 

### Reference
- 2023-1 프로그래밍언어구조론, 조성배 교수님 
- https://www.geeksforgeeks.org/introduction-of-programming-paradigms/