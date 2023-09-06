+++
author = "Soeun"
title = "GDSC mount - BackEnd 탐구(2)"
date = "2023-09-03"
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
## BE 이론

### 테스트 코드란 무엇이고, Unit Test와 Integration Test에 대해서 설명해주세요.

<span style="font-size:120%"><span style="background-color: #EBFFDA">**테스트 코드란?**</span></span>
- 테스트 코드는 소프트웨어의 기능과 동작을 테스트하는데 사용되는 코드이다. 
- 테스트 코드는 소프트웨어의 결함을 찾아내고 수정하는 과정에서 매우 중요하다. 테스트 코드는 개발자가 작성한 코드를 실행하고 예상된 결과가 나오는지 확인하는데 사용된다.

<span style="font-size:120%"><span style="background-color: #EBFFDA">**왜 테스트 코드를 작성해야 할까?**</span></span>
- 내가 무엇을 만들고 있는지 정확하게 파악 가능 : 테스트 케이스를 통해 오류 케이스 또는 코너 케이스를 찾을 수 있으며, 요구사항의 기능적인 항목을 정리할 수 있다.
- 리팩토링을 할 때 부담 덜기 : 테스트 코드가 있으면 코드 수정 후에도 기능이 정상적으로 작동하는지 확인할 수 있다.
- 결합도와 의존도가 낮은 코드 지향 : 테스트 코드 작성을 통해 각 모듈 간 의존성을 낮추는 작업을 할 수 있다.  


<span style="font-size:120%"><span style="background-color: #EBFFDA">**Unit Test**</span></span>
- Unit Test(단위 테스트)란 개별적인 코드 단위(함수, 메서드)가 의도한 대로 작동하는지 확인하는 과정이다. 소프트웨어의 개별 코드 단위를 테스트하여 오류를 발견하고, 이를 수정하여 전체적인 소프트웨어의 품질을 향상시킨다. 
- 이를 위해서는 테스트 케이스를 작성하여, 각각의 코드 단위가 정확한 입력값과 출력값을 반환하는지 확인한다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**Integration Test**</span></span>
  - Integration Test(통합 테스트)란 서로 다른 모듈들 간의 상호작용을 테스트하는 과정이다. 
  - 예를 들어, 신규로 개발한 API 서버 내 DB 호출 함수가 DB의 데이터를 잘 호출하는지 테스트하는 과정이다. 
  - 여러 개의 모듈이 연결된 백엔드 API 웹 어플리케이션의 경우에는 서로 다른 모듈들 간의 상호작용을 테스트하기 위해 각 모듈 단위 테스트를 모두 완료한 뒤, 둘 이상의 모듈을 거쳐서 동작하는 API 테스트 시나리오를 기반으로 통합 테스트를 수행한다. 
  - 통합 테스느는 단위 테스트에 비해 테스트 케이스를 작성하기 어려운 편이며, 더 많은 리소스와 시간이 필요하다. 
  - GUI 환경에서 통합 테스트를 할 수 있는 도구로는 Postman, Selenium, Apache JMeter 등이 있다. 


    ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/280dd23f-8d36-450d-a253-1ea2f387be33)

### CI/CD란 무엇인가요?

<span style="font-size:120%"><span style="background-color: #EBFFDA">**CI/CD**</span></span>
  - CI/CD (Continuous Integration/Continuous Delivery)란 어**플리케이션 개발 단계부터 배포 때까지 모든 단계를 자동화하여 애플리케이션을 더욱 짧은 주기로 사용자에게 제공하는 방법**이다. CI/CD의 기본 개념은 지속적인 통합, 지속적인 서비스 제공, 지속적인 배포이다. 
  - CI/CD는 애플리케이션의 통합 및 테스트 단계에서부터 제공 및 배포에 이르는 애플리케이션의 라이프사이클 전체에 걸쳐 지속적인 자동화와 지속적인 모니터링을 제공한다. 
  <img width="759" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/610100c6-b14f-460e-ad96-daf8d2acb621">

<span style="font-size:120%"><span style="background-color: #EBFFDA">**CI(Continuous Integration)**</span></span>
  - CI는 개발자를 위한 자동화 프로세스인 지속적인 통합을 의미한다. 지속적인 통합이란, 어플리케이션의 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트 되어 공유 레포지토리에 통합히는 것을 의미한다. 
  - CI가 필요한 환경(1) : **다수의 개발자가 git 과 같은 형상관리 툴을 공유해서 사용할 때**
    - 하나의 어플리케이션을 N명의 개발자들이 함께 개발을 할 때 각 기능별로 작업 후 commit 을 날려 repository 에 버전 업데이트를 한다. 이때 공유 레포에는 수 많은 commit 들이 쌓이게 된다. 그럴 때마다 기능별로 빌드/테스트/병합을 하려면 상당히 번거로울 수 있다. 
    - 이 상황에서 **자동화된 빌드&테스트**는 원천 소스 코드들의 충돌 등을 방어할 수 있다. 
  - CI가 필요한 환경(1) : **MSA(Micro Service Architecture) 환경**
    - MSA는 작은 기능별로 서비스를 잘게 쪼개어 개발하는 형태이다. MSA 환경에서는 대부분 Agile(소규모 기능 단위로 빠르게 개발 & 적용을 반복) 방법론이 적용되기 때문에, 기능 추가가 자주 일어난다. 
    - 이때 작은 micro service 의 긴밀한 동작 테스트가 중요해진다. 
    - 위의 상황에서 CI의 적용은 **기능 충돌 방지**를 할 수 있다. 
  - CI의 핵심 목표는 아래 3가지이다. 
    - 버그를 신속하게 찾아 해결
    - 소프트웨어 품질 개선
    - 새로운 업데이트의 검증 및 릴리즈의 시간 단축

<span style="font-size:120%"><span style="background-color: #EBFFDA">**CD(Continuous Delivery/Continuous Depolyment)**</span></span>
  - CD는 Continuous Delivery(지속적인 제공)과 Continuous Depolyment(지속적인 배포)라는 두 가지 의미가 있다.
  - Continuous Delivery는 공유 레포지토리로 자동으로 Release 하는 것이고, Continuous Deployment는 Production 레벨까지 자동으로 deploy 하는 것을 의미한다. 
  - 개발자들에 의해 수정된 코드들을 지속적으로 빌드 및 테스트하여 통합하고, 이렇게 통합된 application을 지속적으로 공유 레퍼지토리에 release하고 product로 배포하는 전체 과정을 CI/CD라고 한다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**CI/CD 에 사용되는 Tools**</span></span>
  - 대표적으로 Jenkins, TeamCity, Bamboo, Gitlab 등이 있다.

### TCP와 UDP의 차이에 대해 설명해주세요.
<span style="font-size:120%"><span style="background-color: #EBFFDA">**OSI 7 계층**</span></span>
    <img width="674" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4c8ebe59-f2b8-40c4-af06-d1a2926ffda6">
  - OSI 7 계층은 **통신이 일어나는 과정을 7단계로 정의한 국제 통신 표준 규약**이다. 
  - 전송 계층(Transport Layer)은 데이터를 보내기 위해 사용하는 프로톨이 있는데, 그 프로토콜이 TCP와 UDP이다. 
  - 전송계층은 송신자와 수신자를 연결하는 통신 서비스를 제공하고 IP에 의해 전달되는 패킷의 오류를 검사하며 재전송 요구 제어등을 담당하는 계층이다.
  - TCP와 UDP는 포트 번호를 이용하여 주소를 지정하는것과 데이터 오류검사를 위한 체크섬 존재하는 두가지 공통점을 가지고 있지만 정확성(TCP)을 추구할지 신속성(UDP)을 추구할지를 구분하여 나뉜다.

<span style="font-size:120%"><span style="background-color: #EBFFDA">**TCP(Transmission Control Protocol)**</span></span>
  - 데이터를 중요하게 생각하여 확실히 주고받고 싶을 때는 ‘TCP’를 사용한다.  TCP는 통신할 컴퓨터끼리 ‘보냈습니다’, ‘도착했습니다’라고 서로 확인 메시지를 보내면서 데이터를 주고받음으로써 **통신의 신뢰성을 높인다**. 
  - TCP는 **연결 지향적 프로토콜**이다.  연결 지향 프로토콜이란 클라이언트와 서버가 연결된 상태에서 데이터를 주고받는 프로토콜을 의미한다. 클라이언트가 연결 요청(SYN 데이터 전송)을 하고, 서버가 연결을 수락하면 통신 선로가 고정되고, 모든 데이터는 고정된 통신 선로를 통해서 순차적으로 전달된다. 그렇기 때문에 TCP는 데이터를 정확하고 안정적으로 전달할 수 있다. 
  - TCP는 호스트간 신뢰성 있는 데이터 전달과 흐름제어를 한다. TCP 프로토콜은 신뢰성 있는 데이터의 전송을 위해 확인작업을 거치는데 TCP는 패킷을 성공적으로 전송하면(ACK) 라는 신호를 날리고 만약에 ACK 신호가 제 시간에 도착하지 않으면 Timeout이 발생하여 패킷 손실이 발생한 패킷을 다시 전송해준다. TCP는 이렇게 데이터를 송신할때마다 확인 응답을 주고받는 절차가 있으므로 통신의 신뢰성이 올라간다..
  - TCP 통신을 위한 네트워크 연결 3-way handshake 으로 한다. 즉, 3 way handshake 방식은 서로의 통신을 위한 관문(port)을 확인하고 연결하기 위하여 3번의 요청/응답 후에 연결이 되는 것이다.
  - TCP 연결을 해제할 때는 4-way handshake 를 사용한다. 
  - TCP는 신뢰성을 중요하게 여기는 file 전송에 쓰인다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**UDP(User Datagram Protocol)**</span></span>
  - UDP는 데이터를 보내면 그것으로 끝이므로 신뢰성은 없지만 확인 응답과 같은 절차를 생략할 수 있으므로 **통신의 신속성을 높인다**. 
  - UDP는 **비연결형 프로토콜**이다. 비연결형 프로토콜이란 연결을 위해 할당되는 논리적인 경로가 없고, 각각의 패킷은 다른 경로로 전송되며, 독립적인 관계를 지니는 것이다. 
  - UDP는 비연결형 서비스로 데이터그램 방식을 제공한다. 따라서 데이터의 전송 순서가 바뀔 수 있다. 
  - UDP는 데이터 수신 여부를 확인하지 않는다. 따라서 TCP보다 신뢰성이 낮지만, 속도가 빠르다. 
  - UDP는 1:1 & 1:N & N:N 통신이 가능하다.
  - UDP는 신뢰성보다는 연속성 있는 전송이 필요할 때 사용하는 프로토콜로 예를 들면, 실시간 서비스(streaming)에 자주 사용된다.

<span style="font-size:120%"><span style="background-color: #EBFFDA">**TCP vs UDP**</span></span>

  ||TCP|UDP|
  |---|---|---|
  |연결 방식|연결형|비연결형|
  |패킷 교환 방식|가상 회선 방식|데이터그램 방식|
  |전송 순서|전송 순서 보장|전송 순서가 바뀔 수 있음|
  |수신 여부 확인|수신 여부 확인|수신 여부 확인X|
  |통신 방식|1:1 통신|1:1 OR 1:N OR N:N 통신|
  |신뢰성|높음|낮음|
  |속도|느림|빠름|

### www.naver.com을 주소창에 입력했을 때, 네트워크 관점에서 발생하는 일을 최대한 자세하게 설명해주세요. 

1. **브라우저에 www.naver.com 을 브라우저 주소창에 입력한다.**
2. **브라우저는 캐싱된 DNS 기록들을 총해 www.naver.com에 대응하는 IP 주소가 있는지 확인한다.** 
     - DNS는 도메인 이름, IP 주소를 한 쌍으로 저장하고 있는 데이터베이스이다. 
     - 브라우저는 DNS 기록은 4가지의 캐시에서 확인한다. 
       - 브라우저 캐시 : 브라우저는 일정 기간 동안 DNS 기록들을 저장하고 있다. 
       - OS 캐시 : 브라우저는 systemcall 을 통해 OS가 저장하고 있는 DNS 기록들의 캐시에 접근한다.
       - router 캐시 : 브라우저가 컴퓨터에서 DNS 기록을 찾지 못하면, 브라우저는 DNS 기록을 캐싱하고 있는 router 과 통신을 해서 찾으려고 한다.
       - ISP 캐시 : ISP는 DNS 서버를 구축하고 있으므로 브라우저는 마지막으로 여기를 확인한다. 
3. **요청한 URL이 캐시에 없으면, ISP의 DNS 서버가 www.naver.com을 호스팅하고 있는 서버의 IP 주소를 찾기 위해 DNS query를 날린다.**
     - www.naver.com 에 접속하고 싶으면 IP 주소를 반드시 알아야 한다. DNS query 의 목적은 여러 다른 DNS 서버들을 검색해서 해당 사이트의 IP 주소를 찾는 것이다. 이러한 검색을 REcursive search 라고 부른다. IP 주소를 찾을 때 까지 DNS 서버에서 다른 DNS 서버를 오가면서 반복적으로 검색하던지 못 찾아서 에러가 발생할 때 까지 검색을 진행한다.
     - 이 상황에서, ISP의 DNS 서버를 **DNS recursor**라고 부르고 인터넷을 통해 다른 DNS 서버들에게 물어 물어 도메인 이름의 올바른 IP 주소를 찾는데 책임을 갖고 있다. 다른 DNS 서버들은 **name server**라고 불린다. 이들은 웹사이트 도메인 이름의 구조에 기반해서 검색을 하기때문이다. 도메인 이름 구조에 기반에서 검색한다는 것은 URL들을 third-level domain, second-level domain, top-level domain으로 나누어서 각 레벨 별로 자신들의 name server 에 쿼리를 날리는 것이다. 
4. **브라우저가 서버와 TCP 연결을 한다.**
     - 브라우저가 올바른 IP 주소를 받게 되면 서버와 연결을 한다. 웹 사이트의 HTTP 요청의 경우에는 일반적으로 TCP 를 사용한다. 
5. **브라우저가 웹 서버에 HTTP 요청을 한다.**
     - TCP 연결이 되었다면, 데이터를 전송하면 된다. 
     - 클라이언트의 브라우저는 GET method request 를 통해 서버에게 www.naver.com 의 웹페이지를 요구한다. 
6. **서버가 request를 처리하고 response 를 생성한다.**
     - 서버는 웹서버를 가지고 있는데, request handler 에게 요청을 전달해서 요청을 읽고 response 를 생성한다. 이 response는 특정한 포맷(JSON,XML,HTML)로 작성한다.
7. **서버가 HTTP response 를 보낸다.** 
     - 서버의 response 에는 요청한 웹페이지, status code, compression type(Content-Encoding), 어떻게 페이지를 캐싱할지, 쿠키, 개인정보 등이 포함되어 있다. 
8. **브라우저가 HTML content 를 보여준다.** 
     - 브라우저는 response 에 있는 HTML content 를 렌더링해서 보여준다. 

### 네트워크 관련 공격 기법을 3가지 이상 조사해주세요. (ex. SQL Injection, XSS, CSRF, SSRF 등...) 
<span style="font-size:120%"><span style="background-color: #EBFFDA">**SQLi(SQL Injection)**</span></span>
- SQL Injection 이란 악의적인 사용자가 보안상의 취약점을 이용하여, 임의의 SQL 문을 주입하고 실행되게 하여 데이터베이스가 비정상적인 동작을 하도록 조작하는 행위이다. 
- 공격 종류
  - <span style="font-size:100%">Error Based SQL Injection</span>   
    - 논리적 에러를 이용한 SQL Injection이다. 로그인 창에 사용자가 아이디와 비밀번호를 입력하면, SQL 쿼리 'SELECT * FROM Users WHERE id = 'Input1' AND password = 'INPUT2' 로 변환되어 아이디와 비밀번호가 맞는지 확인한다. 이때 'SELECT * FROM Users WHERE id = '<span style="color: red">'OR 1=1 /* </span> <span style="color: gray">AND password = 'INPUT2' </span> 처럼 <span style="color: red">'OR 1=1 /* </span> 부분을 삽입한다. 
    - 그러면 OR 1=1 구문을 사용해 WHERE 절을 모두 참으로 만들고, /* 로 뒤의 구문을 주석처리한다.
    - 이 공격은 입력값에 대한 검증이 없으면 발생할 수 있다.
  - Union based SQL Injection
    - Union 은 두 개의 쿼리문에 대한 결과를 통합해서 하나의 테이블로 보여주게 하는 키워드이다. 정상적인 쿼리문에 Union 키워드를 사용하여 injection 에 성공하면, 원하는 쿼리문을 실행할 수 있게 된다. 
  - Blind SQL Injection
    - Blind SQL Injection은 데이터베이스로부터 특정한 값이나 데이터를 전달받지 않고, 단순히 참과 거짓의 정보만 알 수 있을 때 사용한다. 
    - 로그인 폼에 SQL Injection이 가능하다고 가정 했을 때, 서버가 응답하는 로그인 성공과 로그인 실패 메시지를 이용하여, DB의 테이블 정보 등을 추출해 낼 수 있다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**XSS(Cross Site Scripting)**</span></span> 
- 악의적인 사용자가 사이트에 자바스크립트 같은 스크립트를 넣는 기법이다. 공격에 성공하면 사이트에 접속한 사용자는 삽입된 코드를 실행하게 되며, 보통 의도치 않은 행동을 수행시키거나 쿠키나 세션 토큰 등의 민감한 정보를 탈취한다. 다른 웹해킹 공격 기법과는 다르게 사용자를 대상으로 한 공격이다. 
- 공격 종류
  - Stored XSS
    -  사이트 게시판이나 댓글, 닉네임 등 사이트에 스트립트를 삽입하는 방식이다. 공격자는 게시판에 스크립트를 삽입한 후 사용자가 해당 게시글을 클릭하도록 유도한다.
  - Reflected XSS
    - URL 파라미터(특히 GET 방식)에 스크립트를 넣어 서버에 저장하지 않고 그 즉시 스크립트를 만드는 방식

<span style="font-size:120%"><span style="background-color: #EBFFDA">**CSRF(Cross Site Request Forgery)**</span></span>  
- 인터넷 사용자(희생자)가 자신의 의지와는 무관하게 공격자가 의도한 행위(수정, 삭제, 등록 등)를 특정 웹사이트에 요청하게 만드는 공격이다. CSRF 를 통해 공격자는 사용자의 권한을 도용하여 중요 기능을 실행한다.
- 예를 들어, 페이스북 유저의 계정을 사용하여 광고성 글을 올릴 수 있다. 
- CSRF 공격 방지하기 위한 방법
  - Referrer 검증
    - Back-end 단에서 request의 referrer를 확인하여 domain (ex. *.facebook.com) 이 일치하는 지 검증하는 방법이다. 
  - Security Token 사용 (CRSF Token)
    - 우선 사용자의 세션에 임의의 난수 값을 저장하고 사용자의 요청 마다 해당 난수 값을 포함 시켜 전송한다. 이후 Back-end 단에서 요청을 받을 때마다 세션에 저장된 토큰 값과 요청 파라미터에 전달되는 토큰 값이 일치하는 지 검증하는 방법이다. 
  - Double Submit Cookie 검증
    - Double Submit Cookie 검증은 Security Token 검증의 한 종류로 세션을 사용할 수 없는 환경에서 사용할 수 있는 방법이다. 웹브라우저의 Same Origin 정책으로 인해 자바스크립트에서 타 도메인의 쿠키 값을 확인/수정하지 못한다는 것을 이용한 방어 기법이다. 
    - 스크립트 단에서 요청 시 난수 값을 생성하여 쿠키에 저장하고 동일한 난수 값을 요청 파라미터(혹은 헤더)에도 저장하여 서버로 전송한다. 서버단에서는 쿠키의 토큰 값와 파라미터의 토큰 값이 일치하는 지만 검사하면 된다.

<span style="font-size:120%"><span style="background-color: #EBFFDA">**SSRF(Server Side Request Forgery)**</span></span>  
- 서버 측에서 위조된 HTTP 요청을 발생시켜 직접적인 접근이 제한된 서버 내부 자원에 접근하여 외부로 데이터 유출 및 오동작을 유발하는 공격이다. 
- SSRF 공격은 중간에 사용자를 개입시키지 않고 웹 서버 자체를 노린다. 공격자는 악성 HTTP 요청을 서버에 보내는 것만으로 백엔드에 지시해 악성 작업을 수행할 수 있다.
- 공격형태만 보면 위조된 HTTP 요청(Request Forgery)를 이용한 공격이기 때문에 CSRF(Cross Site Request Forgery)와 유사하다고 볼 수 있으나 공격자의 공격이 발현되는 지점이 서버 측(Server Side)인지 클라이언트 측(Client Side)인지의 여부에 따라서 공격 형태가 구분될 수 있다. CSRF가 사용자의 웹 브라우저를 하이재킹하여 사용자로 하여금 악성 요청을 수행하게 만든다면, SSRF는 접근이 제한된 내부환경에 추가 공격(Post-Exploitation)이 가능하기 때문에 공격의 영향도가 높아질 수밖에 없다.


### 클라우드 컴퓨팅이 무엇인가요? GCP, AWS, NCP 등은 어떤 차이가 있나요?
<span style="font-size:120%"><span style="background-color: #EBFFDA">**클라우드 컴퓨팅**</span></span> 
- 클라우드 컴퓨팅은 인터넷을 통해 원격으로 컴퓨팅 자원 및 서비스를 제공하는 컴퓨팅 기술이다. 클라우드 서비스는 필요한 리소스(하드웨어, 소프트웨어, 데이터 저장소 등)을 필요한 만큼 요청하고 제공받는 온디맨드(on-demand) 방식으로 작동한다.
  - 온디맨드(On-Demand) : IT 자원 관리는 클라우드 서비스 제공자가 하고, IT 서비스를 하는 기업은 IT 자원을 사용하는 방식
  - 온프레미스(On-Premise) : IT 서비스를 기업이 자체적으로 보유한 물리적인 서버에 직접 설치해 운영하는 방식
- 클라우드 컴퓨팅은 3가지 모델이 있다
  - IaaS(Infrastructure as a Service, 인프라 기반 서비스)
    - 사용자가 관리할 수 있는 범위가 가장 넓은 클라우드 컴퓨팅 서비스 = 서버에서 제공하는 범위가 가장 작다
    - 사용자가 서버 OS, 미들웨어, 런타임, 데이터, 어플리케이션까지 직접 구성하고 관리 가능
    - AWS의 EC2와 Google 의 Compute Engine
  - PaaS(Platform as a Service, 플랫폼 기반 서비스)
    - IaaS 형태의 가상화된 클라우드 위에 사용자가 원하는 서비스를 개발할 수 있도록 개발 환경(Platform)을 미리 구축
    - IaaS보다는 관리상의 자유도가 낮다
    - Salesforce의 Heroku, Redhat의 Openshift
  - SaaS(Software as a Service, 소프트웨어 기반 서비스)
    - 바로 사용할 수 있는 소프트웨어 자체를 제공하는 것
    - Slack, Microsoft 365, Dropbox


<span style="font-size:120%"><span style="background-color: #EBFFDA">**GCP vs AWS vs NCP**</span></span> 
||AWS|GCP|NCP|
|:---:|---|---|---|
|차이점|운영자 시각에서 설계된 클라우드 플랫폼|소프트웨어 엔지니어 시각에서 설계된 클라우드 플랫폼|
|규모 및 지역|가장 큰 규모와 지역적으로 확장 가능|전세계 규모|한국 내|
|가격 정책|시작 비용이 낮다|자체 네트워크를 보유해서 데이터 전송 비용이 저렴하다.|로컬 마켓에서 AWS, GCP 보다 저렴|
|기술지원|서드파티 도구 및 플램폼 통합에서 강점|인공지능 및 머신 러닝 분야에서 약간의 강점|한국어로 된 문서 및 24시간 기술 지원|
|기능 및 서비스|- Computing(EC2, Lambda, EKS) <br> - Storage(S3, EFS) <br> - Networking(VPC, Route 53) <br> - Big data(EMR, Athena, Redshift) <br> - ML (SageMaker) <br> - Security(IAM, CloudTrail)|- Computing(Google Compute Engine)<br> - Storage(Cloud SQL, Cloud Datastore) <br> - Networking(DNS, VPN, Load Balancing) <br> - Big data(BigQuery) <br> - ML (Vertex AI) <br> - Security(IAM) |한국 내에서 필요한 기능을 제공하는데 강점|
|장점|- 개인도 바로 쉽게 사용할 수 있다<br> - 인터페이스나 API와 관련하여 다양한 표준 기술이 적용되어 있어, 애플리케이션 개발을 진행할 때 용이하다. <br> - 다른 서비스보다 더 긴 기간 동안 서비스를 제공해 왔기에, 사용자도 많고 AWS를 경험한 엔지니어도 많은 편이다. |- AI 관련 서비스나 데이터 분석 기반 서비스 등에 장점 <br> - Gmail이나 Google Map 등 구글이 제공하는 서비스와 같은 인프라를 활용해 클라우드 서비스를 사용할 수 있다. <br> - Cloud SQL 이 Amazon DynamoDB보다 성능이 좋다 <br> - Bigtable 이 Amazon 의 Redshift 보다 성능이 좋다 |한국에서 사용하기 특화되어 있다|

### 캐시 메모리와, 캐시의 지역성에 대해 설명해주세요.
<span style="font-size:120%"><span style="background-color: #EBFFDA">**캐시(Cache)**</span></span> 

- 캐시는 자주 사용하는 데이터나 값을 미리 복사해놓는 임시 장소이다. 
- 캐시는 저장 공간이 작고 비용이 비싼 대신 빠른 성능을 제공한다. 
  ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5d3a941f-7de6-4fd1-8061-e8b784349fe7)
- 캐시에 미리 데이터를 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다. 결국 Cache란 반복적으로 데이터를 불러오는 경우에, 지속적으로 DBMS 혹은 서버에 요청하는 것이 아니라 Memory에 데이터를 저장하였다가 불러다 쓰는 것을 의미한다.
- 원하는 데이터가 캐시에 존재할 경우 해당 데이터를 반환하며, 이러한 상황을 **Cache Hit**라고 한다. 하지만 원하는 데이터가 캐시에 존재하지 않을 경우 DBMS 또는 서버에 요청을 해야하며 이를 **Cache Miss**라고 한다. 캐시는 저장공간이 작기 때문에, 지속적으로 Cache Miss가 발생하는 데이터의 경우 캐시 전략에 따라서 저장중인 데이터를 변경해야 한다.
- 캐시의 필요성은 80%의 결과는은 20%의 원인 때문에 발생 한다는 **파레토의 법칙**에서 근거를 찾을 수 있다. 즉 20%의 요구가 시스템 리소스의 대부분을 차지한다는 것이다. 그렇기 때문에 20%의 기능에 Cache를 이용함으로써 리소스 사용량은 대폭 줄이고, 성능은 대폭 향상시킬 수 있다.
- 모든 데이터를 담기에는 캐시라는 공간이 적다. 그래서 파레토의 법칙에 해당하는 소수의 데이터를 선별해야 한다. 이때 캐시의 **적중률(Hit Rate)를 극대화** 하고, 캐시의 **지역성**을 고려해서 캐시를 선별해야 한다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**캐시 메모리(Cache Memory)**</span></span> 

- 주기억장치에서 자주 사용하는 프로그램과 데이터를 저장해두어 속도를 빠르게 하는 메모리이다. 
  - 캐시는 주기억장치보다 크기가 작다. 
  - 캐시 기억장치와 주기억장치 사이에서 정보를 옮기는 것을 사상(Mapping, 매핑)이라고 한다. 
- 속도가 빠른 장치(CPU 연산)와 느린 장치(메모리 접근) 간의 속도 차에 따른 병목 현상을 줄이기 위한 범용 메모리이다. 
- 메인 메모리와 CPU 사이에 위치한다.
- 캐시를 사용하면 메모리에 접근하는 횟수가 줄어 컴퓨터 처리 속도 향상된다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**캐시의 지역성(Cache Locality)**</span></span> 

- 캐시에 저장할 데이터가 지역성(Locality)을 가져야 한다. 지역성이란 데이터 접근이 시간적, 혹은 공간적으로 가깝게 일어나는 것을 의미한다. 지역성의 종류는 3가지가 있다. 
- 시간적 지역성
  - 특정 데이터가 한번 접근되었을 경우, 가까운 미래에 또 한번 데이터에 접근할 가능성이 높은 것을 말한다.
  - 예를 들어 for문 처럼 반복문을 보자. 계속해서 같은 데이터를 반복적으로 참조할 수 있다. 이럴 때 캐시에 같은 데이터가 들어있다면 효율적일 것이다. 
- 공간적 지역성
  - 현재 메모리 위치에 가까운 명령어나 데이터가 곧 필요할 수 있다는 것을 의미한다. 
  - 예를 들어 원소가 100개인 배열을 탐색하는 프로그램이 있다고 하자. 첫 번째 원소를 탐색하고 나서 순차적으로 두 번째 원소를 탐색할 것이다. 이때 배열은 원소를 메모리에 순차적으로 저장해 둘 것이다. 따라서 메모리 주소를 순차적으로 참조하게 되는 공간적 지역성이 성립된다.
- 순차 지역성
  - 공간 지역성과 함꼐 설명되기도 한다. 데이터가 순차적으로 엑세스되는 경향을 보인다. 프로그램 내의 명령어가 순차적으로 구성된다.



## Reference
- https://www.redhat.com/ko/topics/devops/what-is-ci-cd
- https://mangkyu.tistory.com/15
- https://github.com/SantonyChoi/what-happens-when-KR
- https://devjin-blog.com/what-happen-browser-search/
- https://noirstar.tistory.com/264
- https://itstory.tk/entry/CSRF-%EA%B3%B5%EA%B2%A9%EC%9D%B4%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-CSRF-%EB%B0%A9%EC%96%B4-%EB%B0%A9%EB%B2%95
- https://www.igloo.co.kr/security-information/ssrf-%EC%B7%A8%EC%95%BD%EC%A0%90%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B3%B5%EA%B2%A9%EC%82%AC%EB%A1%80-%EB%B6%84%EC%84%9D-%EB%B0%8F-%EB%8C%80%EC%9D%91%EB%B0%A9%EC%95%88/
- https://joobly.tistory.com/24
- https://zu-techlog.tistory.com/135