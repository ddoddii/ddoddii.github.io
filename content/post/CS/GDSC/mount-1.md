+++
author = "Soeun"
title = "GDSC mount - BE 이론"
date = "2023-08-31"
description = "GDSC Yonsei - BackEnd Mount"
categories = [
    "CS"
]
tags = [
    "GDSC",
    "BackEnd"
]
image = "https://github.com/ddoddii/DSL-23-1-modeling-AuToeic/assets/95014836/d9821a03-2b61-4b89-b5c4-cc409f04fa23"
+++
## GDSC Yonsei
9월부터 GDSC 에서 본격적으로 활동하기에 앞서 백엔드 부분을 더 깊이 탐구하고자 Mount 과정을 진행하고 있다. 아래는 주어진 여러 질문들이고 공부하며 답변을 작성하였다. 

## BE 이론

### VM과 컨테이너의 차이가 무엇인가요? (Docker, Python VirtualEnv 는 어디에 속하나요?)
- VM과 컨테이너가 필요한 이유
  - 컨테이너 및 가상 머신은 어플리케이션을 IT 인프라 리소스(각 로컬 컴퓨터마다 설치되어 있는 OS, 라이브러리 버전 등)로부터 독립적으로 만드는 기술이다. 이것은 모두 배포 기술에 속한다. 컨테이너와 가상 머신을 사용하면 여러 환경에서 독립적으로 실행할 수 있도록 어플리케이션을 완전히 격리할 수 있다. 또 이것은 기본 인프라를 가상화 또는 추상화 하므로 사용자가 신경 쓸 필요가 없다. 두 가지 기술의 차이점은 확장 방식과 이식성에서 나타난다. 
- VM : Virtual Machine (가상머신)
  - VM은 OS까지 설치된다. 

### 토큰 기반 인증과 세션 기반 인증을 비교해주세요. 본인이라면 어떤 방식을 사용할 것 같나요?

### 동기와 비동기, 블로킹과 논블로킹의 차이에 대해 설명해주세요. 또한, 각각을 사용한 예시 또한 설명해주세요. ORM이 무엇인가요? 단점은 없을까요?

### RDB와 NoSQL의 차이가 무엇인가요? 왜 각자 그런 형태를 선택했을까요?

### DB의 인덱스의 개념에 대해 설명해주세요.

### DB의 트랜잭션과, ACID 원칙에 대해 설명하고, Lock에 대해 설명해주세요.

### 동시성과 병렬성의 차이가 무엇인가요? 멀티코어 프로그래밍 관점에서 설명해주세요.

### 테스트 코드란 무엇이고, Unit Test와 Integration Test에 대해서 설명해주세요.

### CI/CD란 무엇인가요?

### TCP와 UDP의 차이에 대해 설명해주세요.

### www.naver.com 을 주소창에 입력했을 때, 네트워크 관점에서 발생하는 일을 최대한 자세하게 설명해주세요. 

### 네트워크 관련 공격 기법을 3가지 이상 조사해주세요. (ex. SQL ### Injection, XSS, CSRF, SSRF 등...) 

### 클라우드 컴퓨팅이 무엇인가요? GCP, AWS, NCP 등은 어떤 차이가 있나요?

### 캐시 메모리와, 캐시의 지역성에 대해 설명해주세요.


## Reference
- https://aws.amazon.com/ko/compare/the-difference-between-containers-and-virtual-machines/
- 
