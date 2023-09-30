+++
author = "Soeun"
title = "[네트워크] Application Layer"
date = "2023-09-23"
description = "네트워크 어플리케이션, Web 과 HTTP"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = ""
+++

## 2.1 Principles of network applications

어떠한 프로그램을 만들면, 그 프로그램은 네트워크-코어에 있는 장치들(라우터, 링크 스위치 등등) 에서 돌아가지 않고 사용자의 컴퓨터에서 돌아간다. 그렇다면 이 어플리케이션 계층에서 돌아가는 프로그램들끼리는 어떻게 소통을 하는 걸까?

프로그래머는 application architecture 를 설계한다. 여기서 두 가지 application architecture에는 client-server architecture 과 peer-to-peer architecture 가 있다. 

- Client-Server Architecture 
	
	서버와 클라이언트가 존재한다. 
	- Server
		- always-on 
		- 고정 IP 주소
		- 스케일링을 하기 위해 data center 를 사용
	- Client
		- 간헐적으로 연결된다 
		- 유동 IP 주소를 가질 수 있다
		- 클라이언트끼리는 직접적으로 소통하지 않는다.
- Peer-to-Peer(P2P) Architecture 
	- always-on 서버가 없고, 호스트끼리 직접적인 소통을 한다. 
	- Self-scalability : 새로운 호스트가 들어오면, 규모가 확장된다. 

### Process Communication

프로세스는 호스트 안에서 돌아가고 있는 프로그램이다. 프로그램을 실행시키면 프로세스가 발생하는데, 프로그램은 1개이지만 프로세스는 여러 개가 될 수 있다. 같은 호스트 내에서 두 개의 프로세스가 소통할 때는 **inter-process communication** 을 사용한다. 다른 호스트에 있는 프로세스 끼리 소통할 때는 **messages 를 교환**한다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Client and Server Process**</span></span>  

Client process 는 소통을 시작하는 프로세스이고, Server process 는 소통을 기다리는 프로세스이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Socket**</span></span>  

프로세스끼리 소통을 할 때는 Socket 을 사용한다. 클라이언트 프로세스는 소켓에게 메세지를  보내고, 서버 프로세스는 소켓을 통해 메세지를 받는다. 소켓은 문이고, 프로세스는 집이라고 생각하면 된다. 

소켓은 application layer 과 transport layer 사이에 인터페이스이다. 그래서 application 과 network 사이의 Application Programming Interface(API) 라고도 불린다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f556e42c-5b43-468d-9abe-9d7db5995159)

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Addressing Processes**</span></span>  

받는 프로세스가 어디 있는지 찾으려면 주소를 알아야 한다. 이때 2가지를 알아야 하는데, (1) 호스트 주소 (2) 받는 프로세스가 호스트 내에 어디 있는지 이다. 

호스트는 **IP address**를 통해 알 수 있다. 호스트 내에서 많은 프로세스가 실행 중일 수 있기 때문에, 특정 프로세스를 구분하려면 **Port number** 가 필요하다. 

### Transport Services available to applications

소켓이 application layer 과 transport layer 사이에 인터페이스라고 했다. transport layer 에서도 소켓을 잘 전달하기 위한 여러가지 프로토콜이 존재한다. 각 프로토콜은 transport layer 가 제공하는 여러 서비스들을 적절하게 취사선택해서 제공한다. 그렇다면 transport layer가 제공하는 서비스에는 어떤 것들이 있는지 보자. 

1. Reliable Data Transfer
	
	데이터가 한 쪽 어플리케이션에서 다른 쪽 어플리케이션에 성공적으로 전달 되는 것을 보장하는 것이다.    
2. Throughput
   
	보내는 프로세스가 받는 프로세스에게 메세지의 bit 들을 얼마나 빨리 전송할 수 있는지(rate)이다. real-time 어플리케이션이면, r bits/sec 의 Throughput 을 요구할 수 있다. 이 요구사항이 있는 어플리케이션은 bandwidth-sensitive application 이라고 한다.  
3.  Timing
	
	보내는 프로세스에서 소켓을 전송한 후 받는 프로세스의 소켓까지 걸리는 시간이 100msec 이하가 되도록 요구할 수 있다. 이것은 동시에 상호작용하는 어플리케이션에게 요구되는 사항이다. 
	
4. Security
	
	보안이 없다면 패킷을 보내는 도중에 해킹 당할 수 도 있다. 

### Transport Services Provided by the Internet

2.1.3 에서는 transport services 가 네트워크에게 제공할 수 있는 서비스들에 대해 알아보았다. 이번에는 실제로 인터넷이 제공하는 서비스들에 대해 알아보자. 2가지 transport protocol 이 있는데, UDP 와 TCP 이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**TCP**</span></span>  
- Connection-oriented service
	  TCP 는 클라이언트와 서버가 연결된 것을 확인 한 후에 데이터를 전송한다. 
- reliable transport 
- flow control
- congestion control 
- does not provide : timing, throughput guarantee , security 

뒷 장에서 더 자세히 배우겠지만, security 를 더 강화한 , TCP 기반으로 만든 Secure Sockets Layer (SSL) 이 있다. SSL에서는 소켓을 보낼 때 암호화를 해서 보내기 때문에 보안이 훨씬 좋다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**UDP**</span></span>  
- connectionless (핸드쉐이킹 과정이 없다)
- unreliable data transfer
- does not provide : timing, throughput guarantee , security , connection setup
   
-> 여기서 의문인게 .. UDP 는 unreliable 하며,  TCP 가 제공하지 못하는 3가지도 제공하지 못한다. 그렇다면 UDP 는 왜 존재할까? 

아래와 같이 스트리밍 서비스와 같이 조금은 결함 (오디오 끊김이라던가) 이 있어도 되지만, 속도는 빨라야 할 때 UDP 를 사용한다. 즉 데이터의 완전함 보다 속도를 더 중시하는 것이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f28c34c2-c0c8-4e39-8d7f-afeb499e3e5d)


## 2.2 Web and HTTP


지금까지는 전송 계층에서의 프로토콜에 대해 알아보았다. 각 프로세스가 소켓을 통해 메세지를 전송한다고 했다. 그러면 이 메세지는 어떻게 구조화되어 있고, 어떤 의미를 가지고 있을까? 이것에 대한 답은 application layer protocol 에 있다.  이 중 Web 어플리케이션에서 쓰이는 프로토콜이 HTTP (HyperText Transfer Protocol) 이다.

### Overview of HTTP

Web page 는 여러 object 들로 이루어져 있다. 이 object 에는 base HTML 파일, JPEG 이미지, 오디오 파일.. 등등이 있다. base HTML 파일은 다른 object 들을 그 object URL 을 사용해서 불러온다. URL 은 두 가지 구성 요소로 이루어져 있다 : 서버의 hostname, object 의 path name

https://www.yonsei.ac.kr/en_sc/intro/greeting_2016.jsp

여기서 http://www.yonsei.ac.kr 이 hostname 이고, en_sc/intro/greeting_2016.jsp
 가 특정 object의 path 가 되는 것이다. 

웹 브라우저(클라이언트) 가 서버에서 웹 페이지를 요청할 때, 브라우저는 HTTP request 를 보낸다. 그러면 서버가 request 를 받아서 처리 후 response 를 브라우저에게 보내준다. 

HTTP는 주로 TCP 를 사용한다. 클라이언트는 TCP connection 을 시작하고, 서버는 클라이언트로부터 온 TCP connection 을 수락한다. 그런 다음, 브라우저와 서버 사이에 HTTP messages 가 교환된다. 그 후, TCP connection 이 끝난다. 중요한 것은, HTTP 는 stateless 하다는 것이다. 즉, 서버는 지난 클라이언트에 대한 접속 정보를 모른다 ! 이것을 보완하기 위해 cookie 라는 개념이 나왔다. 

### Non-Persistent and Persistent Connections

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Non-Persistent Connections**</span></span>

각 request/response 가 각각 새로운 TCP connection 을 통해 보내야 한다. 만약 10개의 object 를 보내고 싶으면, 10개의 새로운 TCP connection 을 만들어야 한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bcaf917e-6c78-4333-b633-c40c01afb4ef)

RTT는 Round Trip Time 으로, 작은 패킷이 클라이언트 -> 서버 -> 클라이언트로 오기 까지 걸리는 시간이다. 한 개의 object 를 받기 위한 HTTP response time 은 2RTT + file transmission time 이다. 하나의 TCP connection 을 만들기 위해 1 RTT 가 필요하고, HTTP request 를 보내고 받는데 1 RTT, 그리고 file transmission time 이 필요하다. 

Non-Persistent Connections 에는 object 당 2RTT 가 필요하다. 이러면 OS가 각 TCP connection 마다 메모리를 할당해주어야 해서 overhead 가 발생한다. 

그래서 HTTP 1.1 부터 Persistent Connections 가 나타났다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Persistent Connections**</span></span>  

서버는 HTTP response 를 보낸 후에도 connection 을 열어둔다. 따라서 다른 Object 를 보내고 싶을 때 TCP connection 을 연결할 필요 없이, 바로 request 를 보낼 수 있다. 

###  HTTP Message Format

HTTP message 에는 2가지 종류가 있다 : request, response

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Request Message**</span></span>  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2a9e18dc-2151-487d-9065-a86fb95937df)


request line 은 가장 첫줄인데, 3가지 field 가 있다 : method field, URL field, HTTP version field

Method field 에는 GET, POST, HEAD, PUT, DELETE 등 HTTP Method 가 들어간다. 

Header Line 에는 Host, User-agent, Accept-Language , Connection 등의 정보가 들어간다. 

<img width="517" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b6116309-25a5-4e84-a34e-5b7c85088e2d">


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Response Message**</span></span>  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/10d7c7ea-976a-4790-9180-74ded98d4efc)

HTTP Response message 의 첫줄은 status line 이다. 여기에는 HTTP version, HTTP status code, phrase 가 들어간다. 

<img width="574" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5740984d-09ab-4282-9d65-a13f8933c7cf">

### User-Server Interaction : Cookies

HTTP 서버는 stateless 라고 했다. 하지만 쇼핑 사이트에서는 클라이언트를 기억해서, 상품 추천을 하고 싶을 수 도 있다. 그래서 HTTP 는 cookies 를 사용한다. Cookies 는 사이트가 유저를 추적할 수 있게 해준다. 

Cookie 는 4가지 구성요소가 있다
- 1) cookie header line in the HTTP response message
- 2) cookie header line in the HTTP request message
- 3) cookie file kept on the user's end system
- 4) back-end database at the web site 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1d1f3de1-daf6-47e4-98c1-cbad31b16a59)

### Web Caches (Proxy Server)

Web caches 의 목적 : origin server 가 없이 client request 를 만족시키는 것 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bb335367-b5cc-48eb-9875-f2272a12d556)

1. 브라우저는 Web cache (Proxy Server) 에서 TCP 연결을 하고, HTTP Request 를 보낸다. 
2. Web cache 는 요청받은 정보가 있으면 브라우저에게 HTTP Response 를 보낸다.
3. Web cache 가 요청받은 정보가 없다면 origin server 에세 TCP 연결을 하고, HTTP request 를 보낸다. 그러면 Origin server 는 Web cache 에게 HTTP response 를 보낸다. 
4. 그 후 Web cache 는 정보를 받은후 local storage 에 저장하고, 브라우저에게 HTTP response message 에 복사본을 보낸다. 

 <span style="font-size:110%"><span style="background-color: #EBFFDA">**왜 Web Caching 을 할까?**</span></span>  

클라이언트 요청의 응답 시간을 줄인다. Institution 의 access link 에 대한 병목 현상을 줄인다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Caching example**</span></span>  
1. 상황 (1)
   
    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/adfba21e-9ed8-4039-9415-329807ac26fc)

2. 상황(2) - access link rate 를 늘렸을 때

    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5ea16696-b95d-4947-8548-0910529ec21a)

3. 상황(3) - Local Cache 를 설치했을 때 

    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/df3aedf1-f25e-4b49-882d-b037c5a8ab7e)

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Conditional GET**</span></span>  

웹 서버 있는 Object 가 로컬 캐시에 복사된 이후의 시점에 변경 될 수 있다. 이 때 사용하는 것이 Conditional GET 이다. HTTP request message 는 조건문들을 담고 있다. (1) GET Method 를 사용하면,  (2) request message 는 If-Modified-Since : 를 header line 에 포함한다. 

1. Proxy cache 는 서버에 request message 를 보낸다. 
    ```
    GET /fruit/kiwi.gif HTTP/1.1 
    Host: www.exotiquecuisine.com
    ```

2. Web server 는 캐시에게 response message 를 보낸다. 

    ```
    HTTP/1.1 200 OK 
    Date: Sat, 3 Oct 2015 15:39:29 
    Server: Apache/1.3.0 (Unix) 
    Last-Modified: Wed, 9 Sep 2015 09:23:24 
    Content-Type: image/gif 
    (data data data data data ...)
    ```

    캐시는 브라우저에게도 이 내용을 보내지만, 이 정보를 로컬에도 저장한다. 

3. 일주일 후, 다른 브라우저가 같은 정보를 요청한다. 캐시에 이 정보가 있지만, 웹 서버에서는 이 내용이 변경되었는지 체크하기 위해 up-to-date Check 를 보낸다. (Conditional GET)
    ```
    GET /fruit/kiwi.gif HTTP/1.1 
    Host: www.exotiquecuisine.com 
    If-modified-since: Wed, 9 Sep 2015 09:23:24
    ```

4. Web server 는 캐시에게 response 정보를 보낸다.

    ```
    HTTP/1.1 304 Not Modified 
    Date: Sat, 10 Oct 2015 15:39:29 
    Server: Apache/1.3.0 (Unix) 
    (empty entity body)
    ```

    정보가 변경되지 않았으니, 로컬 캐시는 자기한테 저장되어 있는 정보를 브라우저에게 보낸다. 만약 변경 되었다면, response 에 `HTTP/1.0 response OK <data>`  메세지를 보낸다. 


