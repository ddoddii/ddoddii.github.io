+++
author = "Soeun"
title = "[Java] 내부 구조 뜯어보기"
date = "2024-02-21"
summary = "자바는 어떻게 실행되고 JVM은 어떻게 동작하는가?"
categories = [
    "CS"
]
tags = [
    "Java"
]
draft=true
+++

{{< alert icon="comment">}}
"Write Once, Run Anywhere"
{{< /alert >}}

Java 의 슬로건은 "Write Once, Run Anywhere" 입니다. 이것이 가능하게 하는 기술이 바로 JRE(Java Runtime Environment)입니다. JRE 내부에는 JVM 과 Java API 등 자바 파일을 실행할 때 도움을 주는 정보들을 포함하고 있습니다. JRE 덕분에 플랫폼(운영체제)에 상관없이 독립적으로 하나의 자바 소스와 하나의 자바 컴파일러를 통해 코드를 실행시킬 수 있게 되었습니다.

## JVM 이란?

JVM이란 _Java 언어와, Java bytecode 로 컴파일 된 다른 언어들도 실행할 수 있게 해주는 환경을 만들어주는 가상 머신_ 입니다. 프로그래머가 작성한 언어를 컴퓨터가 이해하고 실행하기 위해서는 기계어로 컴파일 해야 합니다. Java(\*.java) 를 Java bytecode(\*.class) 로 컴파일 하는 것은 java compiler 가 합니다. Java bytecode 를 실제 기계어로 해석하는 일을 JVM 이 담당합니다.

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/eeaa452d-0330-4843-b9fa-f88ac01035ab">

본 포스팅에서는 JVM의 내부 구조에 대해 깊이 알아보겠습니다.

## JVM 의 동작 방식

JVM의 역할은 클래스 로더를 통해 자바 어플리케이션을 읽어서 자바 API 와 함께 실행하는 것입니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d3f2c914-f9ab-4473-acf8-d780720ca167)

1. 자바 프로그램을 실행하면 JVM은 OS로부터 메모리를 할당 받는다.
2. 자바 컴파일러(javac)가 자바 소스코드(.java)를 자바 바이트 코드(.class)로 컴파일한다.
3. 클래스 로더는 동적 로딩을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area 에 올린다.
4. 실행 엔진은 Runtime Data Area 에 로딩된 바이트 코드를 명령어 단위로 해석하고 실행한다.
5. 이 과정에서 실행엔진에 의해 Garbage Collector 과 쓰레드 동기화가 이루어진다.

## JVM 의 구조

![image](https://github.com/ddoddii/Computer-Science-Study/assets/95014836/2b417674-9775-408c-8bc5-25b877639daf)

JVM 은 크게 **클래스 로더, 실행 엔진, 런타임 데이터 영역**으로 구성되어 있습니다. 실행 엔진에는 인터프리터, JIT 컴파일러, 가비지 콜렉터가 있습니다. 런타임 데이터 영역에는 메소드 영역, 힙 영역, PC 레지스터, JVM 스택, Native Method Stack 이 있습니다.

### 클래스 로더(Class Loader)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9721bc97-550a-482c-a199-de9a7717cc32)

클래스 로더는 JVM 내로 클래스 파일(\*.class) 를 동적으로 로드하고, 링크를 통해 배치하는 작업을 수행합니다. 즉, 로드된 바이트코드들을 엮어서 JVM 의 메모리 영역인 런타임 데이터 영역에 배치합니다.

클래스를 메모리에 올리는 로딩 기능은 한번에 메모리에 올리지 않고, 어플리케이션에서 필요한 경우 동적으로 메모리에 적재합니다.

클래스 파일의 로딩 순서는 3단계(Loading -> Linking -> Initialization) 으로 구성됩니다.

- Loading : 클래스 파일을 가져와서 JVM의 메모리에 로드한다.
- Linking : 클래스 파일을 사용하기 위해 검증한다.
  - Verifying : 읽어들인 클래스가 JVM 명세에 명시된 대로 구성되어 있는지 검사한다.
  - Preparing : 클래스가 필요로 하는 메모리를 할당한다.
  - Resolving : 클래스의 상수 풀 내 모든 심볼릭 레퍼런드를 다이렉트 레퍼런스로 변경한다.

## Reference

- [Chapter 2. The Structure of the Java Virtual Machine](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html)
- [JVM 내부 구조 & 메모리 영역 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8)
