+++
author = "Soeun"
title = "[Java] Java의 특징"
date = "2023-08-28"
summary = "Compile 언어와 JDK , JVM, JRE"
categories = [
    "CS"
]
tags = [
    "Java"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/384bd77d-965b-4d8d-aa98-015cedc6ad55"
+++

## Java 특징
- **Compile vs Interpreted**    
  - Compile(번역) 언어
    - 프로그래밍 언어로 코드를 짜고 나서 그것을 실행하기 전에 미리 컴퓨터가 읽을 수 있는 언어로 번역하는 것
    - 쉽게 말하자면, 코드의 변역본을 갖고 있다가 컴퓨터는 번역본을 바로 읽는 것
    - 컴퓨터가 자국어(0101001..) 로 된 코드를 읽기 때문에 실행 속도가 빠르다.
    - C , C++ , Java 등이 있다.
    - 하지만 여러 OS(window, mac, linux) 등이 있는데, 윈도우에서 컴파일한 C언어를 mac 또는 Linux 에서 돌리지 못한다. 그래서 실행할 컴퓨터 OS 환경에 맞춰 따로따로 compile 해야 한다.
    - Java 는 실행할 컴퓨터에 jvm(java virtual machine)을 깔아서, 어느 OS위에 Compile 해도, 어느 OS 위에서도 실행할 수 있다! Java로 짠 코드는 처음에 JB(Java Bytecode)로 compile 되는데, jvm 은 JB 도 알아듣고, 각 OS 의 기계어도 알아듣기 때문에 문제없이 실행가능하다. 
    - Compile 언어는 문법오류가 있을 때 아예 compile하지 않고, 오류가 뜬다. 그래서 더 안정적이다.
  - Interpreted(통역) 언어
    - 사람이 읽을 수 있는 언어로 짠 코드를 컴퓨터가 알아들을 수 있게 실시간으로 통역하는 것
    - 사람이 짠 코드를 그대로 가지고 있다가, 프로그램을 실행하면 그때그때 통역 프로그램이 실시간으로 통역해주는 언어
    - 개발이 더 간편하지만, 오류에 취약하고 실행이 더 느리다. 문법에 오류가 있어도 실시간으로 실행할 때 알기 때문이다.
    - 대표적으로 python이 있다. 

- **JVM, JDK ,JRE ?**
    ![img](https://github.com/ddoddii/Study-repo/assets/95014836/36643e06-851c-412d-8b96-2494d107b4d9)
  - JVM(Java Virtual Machine)
    - Java 로 compile 하면 JB가 생기는데 JB를 번역해주는 것이 JVM 
    - OS 기계어에 맞추어 JB를 번역해줌 
    - 마치 현지 식당(맥식당, 리눅스식당, 윈도우식당) 에 파견된 현지 주방장 
  - JRE (Java Runtime Environment)
    - JVM, 표준 라이브러리, 각종 설정파일을 포함한다.
    - 표준 라이브러리는 JVM(주방장)이 요리를 어떻게 할 지 알려주는 레시피 북이라고 생각하면 된다. 단순히 '채썰기' 라는 명령어를 compile 하면 JVM은 채써는 방법을 이 라이브러리에서 확인해서 실행한다. 
    - JVM(주방장)이 파견된 현지 식당에 해당한다. 
  - JDK (Java Development Kit)
    - '레시피를 개발하는 회사'에 해당한다.
    - 개발자가 자바로 프로그래밍하는 전 과정을 도와주는 자바 코드 제작 키트
    - Java Compiler(JB로 번역해주는 번역기), Debugger(코드에 문제가 없는지 살핌), JAR도구(compiler 가 번역한 결과물을 실행용 책자로 예쁘게 제본해줌) ,profiler(성능 모니터링) 도 JDK 에 포함된다.
    - JDK 는 여러 회사에서 만든다 (Amazon, Microsoft, Azul ...)





## Reference
- [제대로 파는 자바](https://www.youtube.com/watch?v=iN22AgS_Chk&t=229)