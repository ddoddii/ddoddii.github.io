+++
author = "Soeun"
title = "API 심층탐구"
date = "2023-07-11"
summary = "API에 대한 (거의) 모든 것"
categories = [
    "CS"
]
tags = [
    "API"
]
image = ""
+++

## 1. API 의 정의

> **API 란 Application Programming Interface 의 줄임말이다.** 

- **Interface**
  - 우리가 티비를 키고 싶을 때, 리모콘을 사용해서 전원버튼을 누른다. 여기서 우리는 리모콘과 티비가 어떻게 상호작용하는지는 모르지만, 리모콘에서 어떤 버튼을 눌러야 티비가 켜지는지 알고 있다. 
  - 리모콘과 티비 사이에 정보를 주고 받기 위한 인터페이스 = 전원 버튼 
  - 즉 서로 다른 두 개체 또는 시스템 간의 상호 작용을 담당하는 규칙, 기능 또는 매개체이다. 
  - 인터페이스는 사용자와 시스템 또는 어플리케이션 간의 상호작용을 지원한다.  

- **Application**
  - 어플리케이션은 소프트웨어 프로그램 또는 앱으로, 사용자가 원하는 기능을 수행하는데 사용한다. 

![스크린샷 2023-08-22 오후 4 37 33](https://github.com/ddoddii/skills-for-DS/assets/95014836/264e6d7b-3641-4879-81af-caf1a763b857)

- **API**
  - API 는 일종의 계약이다. API는 어떠한 프로그램을 어떻게 사용하는지, 그리고 이것을 사용하면 어떠한 결과값을 얻는지 알 수 있다.
  - 개발자끼리 서로 편하려고 의사소통 규칙을 통해 간편하게 서로 요청과 응답을 주고받는 것이다.
  - 요즘 API 라고 하면, 대부분 web-based API 를 뜻한다.

- **API 에 대한 예시**
  - API 는 사실 우리가 프로그래밍 언어들 쓰면서 쓰는 수많은 기본 내장 함수들도 포함한다. 예를 들어, text 에 대해 모두 uppercase 를 만들고 싶을때, 그냥 `text.upper()` 라고 하면 된다. 사실 upper() 뒤에 함수가 어떻게 돌아가는지는 모른다 ! 
  - 또, window 와 mac os 에서 파일 시스템의 생김새는 전혀 다르지만, 같은 코드로 파일들을 불러올 수 있다.
  ```python
    import os

    current_dir = os.getcwd()
    for entry in os.listdir(current_dir):
        print(entry)
    ```
   - 모두 약속된 규칙이 있기 때문에 가능한 것이다 !
   - 그럼 내가 텍스트를 넣으면 그 텍스트에 대한 감정을 반환해주는 프로그램을 만들고 싶다면 ? --> API 문서에 맞게 설계하고, endpoint 를 알려주면 text 를 넣어서 감정을 분류해주는 프로그램을 만들 수 있다 !
   - 여기서 내가 생각하는 API 의 중요성이 나타난다. 우리는 여러 프로젝트를 하면서, 내 컴퓨터 혹은 구글 코랩에서만 살아있는 프로젝트를 많이 했다. 그러나 세상 사람들에게 내가 한 프로젝트 결과물을 공유하고 싶다면 ? 여기서 첫 시작점이 API 를 만드는 것이다. 이것이 내가 API 와 서버에 대해 공부를 시작한 이유이기도 하다. 내가 만든 멋진 프로젝트를 공유하고 싶어서 ! 🥳

## 2. REST API

- **Convention 의 필요성**
  - 우리가 음식점에 가서 메뉴를 주문할 때는 생각해보자.
  - 손님 1 : 국물 시원한거 한 뚝배기 주세요 ~
  - 손님 2 : 뚝배기 불고기 주세요 ~
  - 손님 3 : 불고기 뚝배기 주세요 ~
  - 사실 손님1, 손님2, 손님3 모두 같은 것을 지칭하지만 말하는 **"방식"** 이 모두 다르다. 
  - API 에서도 소통하는 의사소통 규칙의 표준화 , 즉 convention 이 필요해졌다. 이것을 제안한 사람이 로이필딩씨로, **REST(REpresentational State Transfer)** 를 제안했다. 이 REST 방식을 따르는 API 를, **RESTful API** 라고 부른다. 

- **REST 하다란 ?**
  - REST 란 어떤 자원에 대해 CRUD (Create, Read, Update, Delete) 연산을 수행하기 위해 URI(Resource)로 요청을 보내는 것으로, Get, Post 등의 방식(Method) 를 사용하여 요청을 보낸다.
  - 여기서 우리는 Web 을 바탕으로 소통하는 API 에 대해서 많이 말하기 때문에, Web 이 어떠한 방식으로 작동하는지를 먼저 알고 갈 필요가 있다.

## 3. Web 의 작동방식 

- **인터넷에 접속할 때를 생각해보자.** 
  - Client 는 인터넷 웹페이지에 대한 정보를 Server 에 요청한다. 이때, Universal Resource Locator (URL) 을 사용한다. 
  - URL 은 http://www.google.com 로 이루어져 있다. 여기서 Http 는 HyperText Transfer Protocol 의 약자이다.  
  - Protocol : request 에 어떻게 respond 할 지 정해놓은 규칙이다. 
  - 브라우저는 HTTP Request 를 생성하는데, 이때 GET method 를 사용한다. GET method 는 이 request 가 오로지 데이터를 수신하겠다고 선언하는 것이다. 
  - Server 는 이 request 를 받아서, Client 가 요청한 정보를 respond 해준다. 
  - 여기서 respond 에 가장 중요한 정보는 Body 인데, 웹페이지에 대한 요청이므로 HTML(Hyper Text Markup Language) 이 들어있다. 
  - Client 는 이 respond 를 받아서, 브라우저에서 웹 페이지에 대한 정보를 렌더링한다. 

![Img](https://github.com/ddoddii/skills-for-DS/assets/95014836/7d140d0a-99c9-4f58-9362-5bc6ed7019d0)

- **HTTP Request Methods**
  - HTTP Method 는 데이터를 다루는 가장 기초적인 방법인 **CRUD** 에 대응한다. 
  - POST : 서버에 데이터를 전송할 때 사용
  - GET : 서버에서 단순히 데이터를 받아올 때 사용
  - DELETE : 서버에서 데이터를 지울 때 사용
  - PUT : 서버에서 데이터를 업데이트 할 때 사용
  
![img](https://github.com/ddoddii/skills-for-DS/assets/95014836/ffa397bd-74c4-48d6-8f9a-5633a56b8cfc)

- **HTTP Response Status code**
  - Server 가 response 를 보낼 때, 받은 요청이 어떻게 처리되었는지 상태를 나타내는 상태코드를 함께 리턴한다. 
  -  Informational Response(요청을 받았으며 프로세스를 계속 진행) : 100 ~ 199
  -  Successful Response (요청을 성공적으로 받았으며 인식했고 수용) : 200 ~ 299
  -  Redirection Messages(요청 완료를 위해 추가 작업 조치가 필요) : 300 ~ 399
  -  Client error responses(요청의 문법이 잘못되었거나 요청을 처리할 수 없다) : 400 ~ 499
  -  Server error responses(서버의 오류로 인해 요청을 수행할 수 없다) : 500 ~ 599

  -  자주 사용되는 상태 코드
     -  200 OK : 요청이 성공적 !
     -  400 Bad Request : 잘못된 문법으로 서버가 요청을 이해할 수 없다
     -  401 Unauthorized : 클라이언트가 미인증 상태여서 요청을 수행할 수 없다
     -  404 Not Found:서버가 요청받은 리소스를 찾을 수 없다

## 4. RESTful API

- **RESTful API 가 되기 위한 조건 ?**
  - REST 형식을 따르는 API 를 RESTful API 라고 하기로 했다. 
  - RESTful API 가 되려면 아래 6가지 조건을 만족해야 한다.

    ![image](https://github.com/ddoddii/skills-for-DS/assets/95014836/4dfb0b3c-fbab-41b2-be6b-512cd0c3c123)

- **REST API 의 작동방식을 보자** 
  - 어떠한 프로그램을 사용하기 위해 Client 가 Server 에 request 를 보낸다고 생각해보자. --> Client - Server Architecture 
  - 여기서 사용하는 protocol 은 HTTP protocol 이고, Stateless 이다. 왜냐면 Server 는 Client 에 대해 아무것도 기억하지 못한다. 만약에 State 를 저장하고 싶다면 (로그인정보와 같은 것) Client 는 이 정보를 매번 request 를 보낼 때마다 Header 에 첨부하여 같이 보내야 한다. --> Statelessness
  -  Server 는 response 를 보낼 때 , Body 에 Json (JavaScript Object Notation) 형식으로 보낸다. 

    ![스크린샷 2023-08-23 오전 12 07 27](https://github.com/ddoddii/skills-for-DS/assets/95014836/3f99ee10-04ea-424c-917e-3b86d1939f15)

## 5. 세상에 존재하는 API 살펴보기

- 그렇다면 우리는 API 로 어떤 것을 만들 것인가 ?
  - API 에 대한 기초적인 지식은 알았으니.. 세상에 존재하는 수많은 API 에 대해 알아보자.

- [Spotify API](https://developer.spotify.com/documentation/web-api/concepts/authorization)
  - Spotify API Docs 에 가보면, 아래 사진과 같이 설명되어 있다. 나만의 어플리케이션을 만들고, Spotify API 를 사용하여 Spotify 에게 정보를 요청하고 받아오는 것이다. 
  - 예를 들어, 대한민국에서 top 30 인기 노래 정보를 요청해서 받아올 수 있다. 
  - 그렇다면 'top 30 인기차트 시각화'와 같은 나만의 어플리케이션을 만들 수 있다 ! 🥳

    ![image](https://github.com/ddoddii/skills-for-DS/assets/95014836/f8023899-e001-4daa-9efc-489b5eb545b8)

## 6. 결론

- 지금까지 API 의 기초 지식에 대해 살펴 보았다. 내가 만든 ~~엄청나게 멋지고 쩌는~~ 프로젝트를 어떻게 하면 세상에 공개할 수 있을까 찾아보다가 가장 쉬운 방법이 API 형태로 만들어서 공개하는 것이라서 개념부터 공부해보았다. 
- 실제 사용할 수 있는 URL 을 만들기 위해서는 FastAPI 등 프레임워크 사용법과, 실제로 배포할 수 있는 서버 (AWS or GCP) 사용법도 같이 알아야 한다. 
- 갈 길이 멀지만 내 프로젝트가 세상에게 공개될 때까지 기록해보겠다. 


### Reference
- [APIs for Beginnsers 2023- How to use an API](https://www.youtube.com/watch?v=WXsD0ZgxjRw)
