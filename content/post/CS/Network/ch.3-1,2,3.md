+++
author = "Soeun"
title = "[네트워크] Transport Layer의 개념과 UDP"
date = "2023-10-03"
description = "Transport Layer - 1"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
slug = "Transport-layer-and-UDP"
+++

이번에는 어플리케이션 계층과 네트워크 계층 사이에 있는 전송계층(transport layer) 에 대해 알아보겠다. 우선 전송계층과 어플리케이션 계층 간의 관계를 보고, UDP 까지 알아보겠다. 
다음 글에서 TCP 기술의 배경과 자세한 작동원리에 대해 알아보겠다. 

## 1. transport-layer service
전송계층은 어플리케이션 계층에서 서로 다른 호스트의 프로세스들이 소통할 수 있게 하는 logical communication 을 제공한다. 전송계층의 프로토콜은 end system 에서 작동한다. 전송계층은 메세지가 네트워크 계층에서 어떻게 전송되는지는 전혀 관여하지 않는다. 

- 보내는 쪽 : 어플리케이션의 메세지들을 segment들로 쪼개고, 네트워크 계층으로 보낸다. 
- 받는 쪽 : segment 들을 합쳐서 메세지로 복원하고, 어플리케이션 계층으로 전달한다. 

전송계층에서 사용하는 프로토콜에는 2가지가 있는데, TCP와 UDP 이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9afa1145-c02d-4eed-a967-d995025e4ec9)

### Transport vs. Network layer
- Network Layer : **호스트** 사이의 logical communication
- Transport Layer : **프로세스** 사이의 logical communication

예시를 하나 들어보자. 집에 있는 남동생이 미국에 사는 사촌에게 편지를 보낸다고 하자. 남동생에게 편지를 받아서 문 앞 우체통까지 넣어주는 일은 부모님이 한다. 그러면 우체부가 우편을 받아서 미국에 있는 사촌의 집앞에 전달한다. 그러면 사촌의 부모님이 우체통에서 편지를 꺼내서 편지를 사촌에게 전달한다. 

이것을 컴퓨터 네트워크의 세계에 대입하면, 아래와 같다. 
- 프로세스 = 남동생, 사촌
- 호스트 = 우리 집, 사촌 집 
- 우편 체계 = network layer
- 우리 부모님 / 사촌네 부모님 = transport layer

### Internet transport-layer protocols
어플리케이션 계층에게 2가지 transport-layer protocol을 제공한다고 했다. 이 2가지가 TCP, UDP 이다. 뒤에서 자세히 설명할 것이기에, 여기서는 간단히 알아보자.

- TCP : reliable, in-order delivery
	- congestion control 
	- flow control : 만약 받는 쪽에서 보내는 쪽에게 용량 때문에 못받겠다 하면 보내는 쪽의 속도를 늦춘다. 
	- connection setup
- UDP : unreliable, unordered delivery


## 2. Multiplexing and demultiplexing

Multiplexing 과 demultiplexing 은 네트워크 계층에 의해 제공되는 host-to-host 전달 서비스를 process-to-process 전달 서비스로 확장한 것이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8ebdc014-09e3-4e5d-a1ca-e87e8fb29eef)

각 프로세스는 소켓을 통해  segment를 전달하고, 전달 받는다. 각 소켓은 구별하기 위한 식별자가 있다. 이 식별자는 소켓이 TCP, UDP 소켓인지에 달려있다. 

만약 받는 쪽에 여러 개의 프로세스가 돌아가고 있다면, 보내는 쪽에서 어느 프로세스에 패킷을 전달해야 할 지 알아야 한다. 전송 계층의 segment를 정확한 소켓에 전달 받는 것을 <span style="background-color: #FEFBD1">**demultiplexing**</span> 이라고 한다. 보내는 쪽에서 여러 개의 메세지 청크들을 모으고, 캡슐화하고, 네트워크 계층을 통해 보내는 것을 <span style="background-color: #FEFBD1">**multiplexing**</span> 이라고 한다. 


그러면 이 과정이 호스트 내에서 어떻게 이루어지는지 보자.  전송 계층의 multiplexing 은 (1) 고유한 식별자가 있는 소켓 , (2) 어느 소켓으로 가야하는지를 나타내는 segment의 필드 가 있어야 한다. (1) 은 source port number field, (2) destination port number field 이다. 

demultiplexing 과정에 대해 보자. 호스트의 각 소켓은 port number 로 구분된다. 호스트에 segment가 도착하면, 전송 계층은 destination port number 를 확인하고, 이 segment 를 그에 맞는 소켓으로 보낸다. 소켓을 통과한 segment 는 후에 온전한 메세지 형태로 복원된다. 

전송 계층에서 전달되는 segment은 다음과 같은 형태이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4d9520f5-f16c-4697-892d-c0f948f1ed38)

### Connectionless demultiplexing

UDP 소켓을 통과할 때 과정을 보자. 호스트A 가 19157 UDP Port 를 통해 호스트B의 46428 UDP Port 에 메세지를 전송한다고 하자. 호스트A 의 전송 계층은 어플리케이션 데이터, source port # (19157), destination port # (46428) , 헤더 정보를 포함한 segment 를 만든다. 이 segment를 네트워크 계층에 보낸다. 네트워크 계층은 이 segment 를 IP datagram 으로 캡슐화하고, 목적이에 보낸다. 이 segment 가 호스트B에 도착하면, 전송계층은 이 segment 를 검사해서 어느 소켓으로 보낼지 결정한다. 

<span style="background-color: #FEFBD1">**UDP 소켓**</span>은 오직 <span style="background-color: #FEFBD1">**2개의 tuple (목적지 IP, 목적지 port number) 로도 식별**</span> 할 수 있다. 그렇지만 source port number 도 같이 보내는데, 이것은 에러가 떴을 시 리턴하는 주소값으로 사용한다. 하지만 에러가 떠도 호스트에서는 데이터를 재전송하지 않는다. 그냥 에러가 떴음을 인지하기만 한다. 

도착지 IP 주소를 이용하지 않기 때문에, 같은 목적지 port number 를 가진 IP datagram 은 같은 소켓에 들어간다. 

### Connection-oriented demultiplexing

<span style="background-color: #FEFBD1">**TCP 소켓**</span>이 UDP 소켓과 다른 점은 <span style="background-color: #FEFBD1">**4개의 tuple (보낸곳 IP, 보낸 곳 port number, 목적지 IP, 목적지 port number) 로 식별**</span>한다는 점이다. TCP segment 가 목적지에 도달하면, 호스트의 전송계층은 이 4가지 정보를 모두 이용하여 정확한 소켓을 찾아낸다. 

보낸곳의 IP 주소도 같이 이용하기 때문에, 같은 목적지 port number 를 가진 IP datagram 이더라도 보낸 곳의 IP 주소가 다르면 다른 소켓으로 들어간다.

## 3. connectionless transport : UDP

우선 UDP 부터 알아보자. UDP 는 정말 최소한의 노력만 한다. 정보가 제대로 전송되었는지 확인하지 않는다. 그래서 전송한 데이터가 유실되어도 어쩔 수 없다. 

UDP 는 connectionless 하다. 즉 , 연결을 하는 handshaking 과정이 없다. 

위의 모든 단점들에도 불구하고 UDP 를 선택한다면, 그 이유는 무엇일까?

- **Finer application-level control over what data is sent, and when**

	UDP 에서는, 어플리케이션 계층에서 전송 계층으로 데이터를 보내면, 이 데이터가 바로 UDP segment 을 이용해서 보내진다. 반면, TCP 에서는 congestion control mechanism 이 있다. 그래서 데이터를 보낼 때 혼잡한지 아닌지도 고려한다. 그래서 지연이 발생할 수 있다. 따라서 지연이 안되는 것이 중요하고, 약간의 데이터 손실이 허용 가능한 멀티미디어 스트리밍 서비스에서 UDP 를 사용한다. 

- **No connection establishment**

	UDP 는 보내는 쪽과 받는 쪽이 연결을 하지 않는다. DNS가 UDP 를 채택하는 이유이다. DNS가 TCP 를 썼으면 훨씬 느렸을 것이다. 
	
	구글에서도 UDP 에서 발전된 형태인 **QUIC protocol** 을 사용한다. 이것은 UDP 기반으로, 신뢰성을 좀 더 강화한 프로토콜이다. 이것은 TCP 기반으로 HTTP 소통을 할 때보다 훨씬 빠르다. 
	
- **No connection state**

	TCP 는 connection state 를 지속한다. 하지만 UDP 는 상태를 저장하지 않기 때문에, 더 많은 클라이언트들과 연결할 수 있다. 
  
- **Small packet header overhead**

	TCP segment 는 헤더의 사이즈가 20byte 이고, UDP 의 헤더는 8 byte 이다. 

하지만 치명적인 단점은, UDP 는 congestion control 이 안된다는 점이다. 만약 모두가 고화질의 영상을 스트리밍 하려고 하면 어떻게 될까? 혼잡도가 증가하고, 실제 고화질의 영상을 받는 사람은 소수일 것이다. 따라서 요즘은 모든 소스 (UDP 포함) 의 congestion control 을 할 수 있도록 하는 새로운 메커니즘을 도입했다. 

그리고, UDP 에서도 신뢰성 있는 연결을 하는 것이 **가능**하다. 만약 어플리케이션 계층에서 신뢰성을 확인하는 작업이 이루어진다면 가능하다. 구글의 크롬 브라우저에서 UDP 기반의 QUIC 프로토콜을 사용한다고 했다. 이 부분에 대해서는 다른 글에서 더 다루어 보겠다. 

### UDP Segment Structure

UDP 헤더는 각각 2byte 로 이루어진 3개의 field 만으로 구성된다. length 는 UDP segment 에 있는 모든 byte (header + data) 이다. checksum 은 받는 쪽의 호스트에서 segment 를 받는 데 에러가 있었는지를 나타낸다. 

여기서 의문이 들 것이다. 아니 분명 UDP 는 받는 쪽이 제대로 데이터를 잘 받았는지 확인하지 않는다고 했는데, 에러를 나타내는 것이 헤더에 포함되어 있다니..??? 아래 checksum 이 어떤 역할을 하는지 알아보자. 

<img width="321" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6a9174ab-a208-47c7-84f9-5fe4d13ff43e">

<span style="font-size:110%"><span style="background-color: #EBFFDA">**UDP Checksum**</span></span>  

- Goal : 전송된 segment 에 에러(e.g. flipped bits) 가 있는지를 탐지한다. 
- 보내는 쪽
	- segment 내용을 16-bit 정수로 취급한다.
	- checksum : segment 내용들의 합이다. 
	- 보내는 쪽은 이 checksum 을 UDP checksum field 에 넣는다. 
- 받는 쪽
	- 받은 segment 의 checksum 을 계산한다. 
	- 계산한 checksum 과 UDP checksum field 값이 일치하면 에러가 없는거고, 일치하지 않으면 에러가 발생한 것이다. 

link-layer 프로토콜이 에러 체킹 작업을 하는데, 왜 UDP 에서도 checksum 을 통해 에러 확인을 할까? 

왜냐하면 모든 link layer 가 에러 체킹을 한다는 보장이 없기 때문이다. 그리고, 만약 segment 가 link 를 통해 성공적으로 전송이 되었다고 해도, 이 segment 가 라우터의 메모리에 저장되면서 bit error 가 발생할 가능성이 있기 때문이다. 따라서 UDP 는 end-end basis 의 에러 체킹을 한다. 이것은 시스템 디자인의 'end-end principle' 을 따른 것이기도 하다.  

UDP 가 에러 체킹을 하나ㄷ고 해도, 에러를 다시 회복하기 위해 아무것도 하지 않는다. 어떤 UDP 의 형태는 망가진 segment 를 버리기도 하고, 그냥 어플리케이션에 에러만 띄워주기도 한다. 

다음 글에서는 TCP 에 대해 더 자세히 알아보겠다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 