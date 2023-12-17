+++
author = "Soeun"
title = "[Java] Scanner 개념과 사용시 주의할 점"
date = "2023-10-20"
summary = "Scanner : java.util.NoSuchElementException 에러와 해결방법"
categories = [
    "CS"
]
tags = [
    "Java"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/384bd77d-965b-4d8d-aa98-015cedc6ad55"
draft = true
+++

우테코 1주차 숫자야구 미션에서, 테스트 코드를 돌렸을 때 생긴 에러와 해결방법을 정리해보고자 한다. 

## Scanner : java.util.NoSuchElementException 에러

ApplicationTest 에서 `게임종료_후_재시작` test 를 실행시키자, 
아래의 `java.util.NoSuchElementException` 가 떴다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f1787015-5afa-4f55-ae0a-c9acd22216f3)

구글링 한 결과, Scanner 객체 사용 오류로 발생히는 것 같았다. '1'을 입력할 때, Scanner 어딘가에서 오류가 나서, 입력을 못받아서 NoSuchElementException 이 발생한 것이 아닐까 생각했다.

나는 숫자를 입력해주세요 : 뒤에 입력을 받을 때, UserInput 클래스에서 Scanner 객체를 생성해서 입력을 받았다. 그 후에 게임 종료 / 진행을 결정할 때 UserDecision 클래스에서도 또 Scanner 객체를 생성해 사용자의 입력을 받았다. 

stackoverflow 를 뒤져본 결과, Java에서 `Scanner` 객체를 사용할 때 한 `Scanner` 객체가 `System.in`을 닫은 다음 다른 `Scanner` 객체가 동일한 `System.in`을 사용하려고 시도하면 `java.util.NoSuchElementException`이 발생할 수 있다고 나왔다. 왜냐하면, 
`System.in`은 JVM 내에서 공유되는 단일 리소스이다. 때문에 한 번 닫힌 후에는 다시 열리지 않는다. `Scanner`가 종료될 때 `System.in`도 함께 닫히므로, 여러 `Scanner` 객체를 만들어 사용할 때는 `System.in`을 닫지 않도록 주의해야 한다.


## 해결방법
그래서 우선 해결방법은, sharedScanner 을 main 에 두고, 각자 클래스에 주입해줬다. 
 
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b1f665c9-4ae8-4b27-85b6-67dcc4760227)


## Scanner ?

따라서 Scanner 에 대해 더 깊이 공부해보고자 한다.













## Reference
- https://stackoverflow.com/questions/50748075/exception-in-thread-main-java-util-nosuchelementexception-at-java-util-scanner
- https://hannamedia.tistory.com/26
- https://semicolon-dev.tistory.com/70