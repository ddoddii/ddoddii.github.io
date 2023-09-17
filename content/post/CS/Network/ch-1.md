+++
author = "Soeun"
title = "[네트워크] Computer Netwoks and the Internet"
date = "2023-09-16"
description = "네트워크 기초"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = ""
+++


## 1.1 What is the Internet ?

### Nuts and Bolts Description

> Internet is network of networks. 

인터넷은 전 세계에 있는 수백만대의 컴퓨터를 연결하는 것으로, 컴퓨터 네트워크라 불렸다. 하지만 점차 아이패드, 스마트워치, 센서 등 인터넷에 연결되는 기기들이 많아지며, '컴퓨터' 네트워크라는 단어는 점점 올드해졌다. 연결되어 있는 모든 기기들은 host 또는 end system 이라 불린다. 

End system 들은 communication link 또는 packet switches 들을 통해 연결되어 있다. communication link는 coaxial cable, copper wire, radio network 등 여러가지 유형이 있다. 이것들은 모두 다른 전송 속도(bits/second) 를 가지고 있다. 하나의 end system 이 다른 end system 으로 데이터를 전송할 때, 보내는 쪽에서는 데이터를 여러 조각으로 나누고, 각 조각에 Header 를 붙인다. 이것은 packet 이라고 불린다. 이 packet 이 네트워크를 통해 받는 end system 쪽으로 전송된다. 

Packet switch 는 들어오는 링크에서 packet 을 받고, 나가는 링크로 packet 을 전송한다. Packet switch 에는 여러가지 형태가 있는데, 요즘 인터넷에서 가장 많이 쓰이는 것은 routers 과 link-layer switches 이다. 두가지 모두 packet 들을 최종적인 목적지로 보낸다. Packet 들이 네트워크 안에서 전송되는 길을 route 또는 path 라고 한다. 

End system 은 Internet Service Providers(ISP)를 통해 인터넷에 접속한다. ISP 도 회사 ISP, 대학교 ISP 등 여러 종류가 있고, 티어로도 나누어져 있다. 각 ISP 들은 packet switches 와 communication link 로 이루어진 네트워크 자체이다. 가정에서는 인터넷에 접속 할 수 있게 cable modem 또는 DSL 을 제공한다. 

End system, packet switches 와 인터넷의 모든 부분들은 인터넷 내에서 정보를 전송하고 받는데 필요한 protocol(규칙)을 운영한다. Transmission Control Protocol(TCP) 와 Internet Protocol(IP) 가 가장 중요한 프로토콜 중 2가지이다. 

### Service Description

> Internet is an infrastructure that provides services to applications. 

인터넷을 어플리케이션에게 서비스를 제공하는 인프라로 볼 수도 있다. 이메일, 웹 서핑, 넷플릭스 보기 , 줌 회의 하기 등 여러가지 어플리케이션이 제공하는 서비스가 있다. 가장 중요한 것은 이 어플리케이션이 end system (내가 보는 컴퓨터)에서 돌아가지, 네트워크 중간의 packet switch 에서 돌아가는 것이 아니라는 거다. Packet switch 는 end system 끼리 정보를 전송하는 것을 도와주지만, 어플리케이션과는 관계가 없다. 

내가 만약 정말 기발한 아이디어가 있다면, 이것을 어떻게 인터넷 어플리케이션으로 만들 수 있을까? 이 어플리케이션은 end system (ex. 개인 컴퓨터) 에서 작동하기 때문에, 난 개인 컴퓨터에서 돌아갈 수 있게 Java 또는 python 으로 개발할 것이다. 그러면 하나의 end system 에서 돌아가고 있는 프로그램이 다른 end system 에게 인터넷을 통해 데이터를 전송할 수 있을까? 

인터넷에 연결되어 있는 end system  은 socket interface 라는 것을 제공한다. 이것은 하나의 end system 에서 돌아가고 있는 프로그램이 다른 end system 에 데이터를 전송하기 위해 어떻게 인터넷 인프라에게 요청하는지에 대한 정보를 담고 있다. 즉, 인터넷은 하나의 프로그램이 정보를 전송하기 위한 규칙인 socket interface 가 있다. 

### 여기서 더 알아볼 용어
- Packet switching
- TCP/IP
- Routers
- Communication Links
- Distributed Application

### What is a Protocol ?
<img width="716" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4fa9527d-b536-40b8-8bed-5621e7196d87">
사람 사이에도 의사소통 시에 암묵적인 규칙이 있다. 기계들이 소통할 때도 마찬가지이다. Protocol 은 2개의 communication entities 가 하나의 task 를 수행하기 위해 공통으로 따르는 규칙이다. 

> A protocol defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event.

## 1.2 The Network Edge

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3e412218-551f-40f9-bb9b-02ef603e1146)

컴퓨터와 다른 기기들을 인터넷의 가장자리 에 있어서 end system 이라고 부른다. 여기서 end system 은 host 라고도 불린다. Host 는 client 와 server 로도 나뉘는데, client 를 우리가 사용하는 데스크탑, server 는 웹 페이지를 제공하는 더 강력한 기계라고 생각하면 된다. 

### Access Networks

Access Network 는 end system 에서 첫번째 router (edge router) 로 연결시켜주는 물리적인 네트워크이다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Home Access : DSL, Cable, FTTH, Dial-up , Satellite**</span></span>  

- **DSL (Digital subscriber line)**

    가정에서는 전화 접속을 제공해주는 회사(telco)로부터 DSL 인터넷 접속을 할 수 있다. 그래서 telco 가 ISP 역할도 한다. 가정에서 DSL modem 은 기존에 존재하는 전화선을 사용한다. 그래서 이 전화선을 통과하는 정보는 인터넷으로 가고, 음성은 전화넷으로 간다. 따라서 이 전화선은 정보와 음성을 동시에 전송하는데, 다른 주파수로 전송한다. 이것은 frequency-division multiplexing 이라고 한다. 

    하지만 DSL 방식은 Central Office(CO)와 가정의 거리가 멀면 전송 속도가 급격하게 느려진다. 

    <img width="766" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9976853b-a737-4953-a13b-48f3159cb852">

- **Cable Network**

    DSL 이 가정의 전화선을 사용한다면, cable network 는 가정의 cable TV 에 연결되어 있는 선을 사용한다. 그래서 cable TV 를 제공하는 회사가 ISP 가 된다. Cable Head End 에서 각 동네 단위까지 Fiber cable 을 사용하고, 각 가정까지는 coaxial cable 을 사용한다. 그래서 이것을 hybrid fiber coax (HFC) 라고도 부른다. 

    각 가정에서 head 로 보내는 packet 은 upstream channel 을 따라 head 로 가고, head 에서 가정으로 보내는 packet 은 downstream channel 을 따라 가정에 도착한다.  각 가정까지 shared cable 을 쓰기 때문에(shared broadcast medium), 사용자가 몰리면 속도가 느려진다. 각 채널들을 공유하기 때문에, 충돌을 방지하기 우해서는 distributed multiple access protocol 이 필요하다. 


    ![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f4c40134-b4b5-4084-9d8a-9f0b4dea030c)

- **FTTH (Fiber-to-the-Home)**

    요즘은 fiber 케이블을 각 가정까지 연결하는 FTTH 를 많이 쓰고 있다. FTTH 는 Central Office 에서 각 가정까지 다이렉트로 광케이블을 사용해서 연결하는 개념이다. 그래서 속도가 무지 빠르다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Enterprise access networks (Ethernet and Wifi)**</span></span>


요즘은 end system 에서 edge router 까지 무선으로 연결하는 LAN (local area network) 이 많이 쓰이고 있다. 이더넷은 대표적인 LAN 기술이다. 이더넷 사용자들은 이더넷 스위치에 연결하기 위해 구리선을 사용한다. 각 end system 과 연결된 이더넷 스위치는 더 큰 인터넷에 연결된다. 

요즘은 개인 컴퓨터가 많아짐에 따라 무선으로 연결하는 Wifi 가 더 많이 사용된다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Hosts : sends packets of data**</span></span>

Host 는 보내고자 하는 정보를 작은 조각들 (packet) 으로 나눈다. 각 패킷 은 L bits 길이이다. 그러고, 이 패킷을 transmission rate 가 R 인 access network 으로 보낸다. 

그렇다면 packet transmission delay = 링크를 통해 L-bit 의 패킷을 보내기 위한 시간 = L / R (sec) 이다. 

### Physical media

|guided media|unguided media|
|---|----|
|solid medium (fiber-optic cable)|wireless LAN, digital satellite channel|

- Coaxial cable 
	- 양방향

- Fiber optic cable
	- 광 케이블 - 속도가 굉장히 빠르다.
	- 데이터 손실이 적다.
	- 전기의 영향을 덜 받는다.
- Radio
	- 물리적인 wire 가 없다.
	- 양방향 
	- 장애물을 만나면 영향을 받는다. 
	- Radio Link Types
		- LAN (Wifi)
		- wide-area
		- satellite

## 1.3 Network Core

네트워크 코어는 서로 연결되어 있는 라우터와 패킷 스위치들이다. 

### Packet Switching
각 end system 은 정보를 주고 받는다. 이 때, 정보를 패킷이라는 작은 단위로 쪼갠다. 각 패킷들은 송신자로부터 시작해 네트워크 안에서 communication link 와 packet switches 를 따라 수신자에 도착한다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Store and Forward**</span></span>

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7250301c-a46f-462f-a99e-178d7eba2356)
L-bit 의 각 패킷을 R bps 를 가진 링크를 통해 전송하는데 L/R 초가 걸린다. 라우터가 다른 링크로 패킷을 전송하기 위해서는 전체 패킷을 받아야 한다. 전체 패킷을 다 받기 전까지 일부분을 저장하는 것을 패킷 조각들을 buffer 한다고 한다. 이것을 라우터의 store-and-forward 라고 한다. 

하나의 패킷을 소스에서 목적지까지 전송하기 위해 걸리는 시간을 측정해보자. 두 개의 링크가 있고, 사이에 하나의 라우터가 있다고 하자. 이때 하나의 패킷이 전부 라우터에 도착하는 시간은 L/R 초이다. 그리고 라우터가 이 패킷 전체를 다시 링크를 통해 목적지로 전송하는 시간은 L/R 초이다. 그래서 전부 2L/R 초가 걸린다. 

N개의 링크가 있다고 했을 때, end-to-end delay 는 N * L/R 초이다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Queuing Delays and Packet Loss**</span></span>

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5c9b7077-6836-4946-b463-47a288fd07d0)

각 패킷 스위치는 여러개의 링크가 연결되어 있다. 각 링크에 대해서 패킷 스위치는 output queue 를 가지고 있다. 이것은 그 링크로 패킷을 보내기 전 패킷들을 저장하는 장소이다. B 패킷이 패킷 스위치에 도착해서 특정 링크로 보내고자 하는데, A 패킷을 그 링크로 전송하는 중이라면 B 패킷은 패킷 스위치에서 기다려야 한다. 이 지연을 queuing delay 라고 한다. 

만약 arrival rate 가 링크의 transmission rate 보다 크면 패킷이 기다리게 된다. 이 때 패킷 스위치의 메모리가 꽉 차면 패킷이 드롭될 수 있다.이것은 정보의 손실을 의미한다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Two key network-core functions**</span></span>

- Routing

	패킷이 목적지에 도달하기 위한 루트를 결정한다. 

- Forwarding

	패킷을 라우터의 인풋에서 라우터의 아웃풋으로 옮긴다. 

### Circuit Switching

> End-to-end resources allocated to, reserved for "call" between source & destination.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9ce97f15-19ea-4fea-accf-f70b4498103f)

데이터를 네트워크를 통해 전송하는 두가지 접근법이 Circuit Switching 과 Packet Switching 이다. 
Circuit Switching 에서는 데이터가 예약된 링크를 따라 움직인다. 이 예약된 circuit 는 공유되지 않고, 독점한다. 대표적인 예시로는 유선전화가 있다. 

Circuit Switching 을 사용하면 퍼포먼스가 보장되어 있다. 이미 circuit 이 예약되어 있기 때문에, 전화를 걸 때 송신자는 수신자에게 데이터를 보장된 rate 에 제공할 수 있다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Multiplexing in Circuit Switching Networks**</span></span>

- FDM (Frequency-Division Multiplexing)
	![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5cf6e37e-5a83-4de7-a56d-5bea41ea0011)
	
	FDM 에서는 frequency 가 각 사용자들에게 분할 된다. 예시로는 라디오에서 특정 주파수가 다른 라디오 채널에 할당되는 것이 있다.

- TDM (Time-Division Multiplexing)
	![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c78d1ae5-56ce-4778-97c5-6ce2c6676b52)
	Time 을 사용자마다 고정 시간으로 나눈 것이다. 


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Packet Switching vs. Circuit Switching**</span></span>

패킷 스위칭을 사용하면 서킷 스위칭보다 더 많은 사용자들이 네트워크를 사용할 수 있다 !

ex. 1 Mb/s link, 각 유저는 active 할 때 100kb/s 이고, 전체 시간의 10%만 active 하다.

이때, 서킷 스위칭은 오직 10명의 유저만 네트워크를 쓸 수 있다. 왜냐하면 1Mbps / 100 kbps = 10 users 이고, 서킷 스위칭은 무조건 성능을 만족해야 한다. 하지만 각 사용자는 10%만 active 하니까 나머지 시간은 네트워크 리소스를 낭비한다. 

패킷 스위칭을 사용하고, 35명의 유저가 있다고 하자. 그렇다면 35명 중 10명이 동시에 active 할 확률은 0.0004 보다 작다. 그러므로 더 많은 유저가 네트워크를 이용할 수 있다 !

하지만 패킷 스위칭이 항상 무조건 좋은 것은 아니다. 패킷 스위칭은 bursty data (On-Off 가 확실한 데이터) 에 대해서 좋다. 그리고 circuit 를 예약하는 과정이 없기 때문에 훨씬 단순하다. 하지만 정말 사용자들이 동시에 접속 할 시 지연이 발생한다. 패킷 스위칭에서 서킷 스위칭같은 성능을 어떻게 보장할 지가 중요한 관건이다. 

### Network of Networks 

아까 각 end system 이 인터넷에 접속할 수 있도록 하는 것이 ISP 라고 했다. 우리나라만 해도 kt, skt , U+ 등 이 있는데, 그렇다면 전 세계의 ISP 는 어떻게 연결되어 있을까? 각 ISP 를 모두 연결 ? 하지만 이것은 O(N^2) 의 연결 수를 의미한다. 현실적으로 불가능하다. 

그래서 ISP 는 계층적으로 연결된다. 우선 각 access net 을 연결한 Regional net 이 있고, 거대한 global ISP (Tier 1) 들이 여러개 있다. Global ISP 끼리는 Peering Link 또는 IXP (Internet Exchange Point) 로 연결되어 있다. 

요즘은 Google, Microsoft 와 같은 content provider network 도 있어서, 그들의 네트워크를 end users 에게 제공한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/81fe7e25-4e28-4591-acd9-9a731f6abe28)

## 1.4 Delay, Loss and Throughput in Packet-Switched Networks

### Delay

지연은 라우터 버퍼에서 패킷이 기다릴 때 발생한다. 이것은 패킷이 도착하는 속도가 링크의 전송 속도보다 빠를 때 발생한다. 
![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1044a6e4-f870-495f-b90e-04c7e8feae68)

지연에도 4가지 종류가 있다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/75a5ba4a-0093-43f8-ac41-6152607b1d12)

- Nodal processing delay
	- 라우터 A 에서 라우터 B 로 패킷을 전송한다고 하자.이때 라우터 A 에서는 bit error 를 확인하고, 패킷의 헤더를 검사하고, 패킷을 보낼 output link 를 결정한다. 이때 생기는 지연이 Nodal processing delay 이다. 
	- 이 지연은 주로 microsecond 보다 작다. 
- Queueing delay
	- 패킷이 링크로 전송되기 전 라우터 버퍼에서 기다리는 지연이다. 이 지연은 전송되기를 기다리고 있는 패킷의 수가 몇개인지에 달려있다. 
- Transmission delay
	- 라우터가 패킷을 전송하기 위해서는 패킷의 전체가 도착해야 한다. 패킷의 길이가 L-bit 이고, 링크의 전송 속도가 R Mbps 일 때, Transmission delay 는 L/R 초이다. 
- Propagation delay 
	- 패킷이 링크로 push 되었을 때, 라우터 B (최종 목적지) 로 가야한다. 링크의 물리적 길이를 d 라고 하고, s 를 링크의 propagation speed (초당 패킷을 몇 m 를 보내는지) 라고 하면, Propagation delay 는 d/s 이다. 

d_trans 와 d_prop 는 다른 개념이다 !! 구분하자 ⭐️

### Queuing delay 
위 모든 지연 중에서 가장 중요하고 복잡한 지연이 Queueing delay 이다. 다른 지연들과 다르게, Queueing delay 는 패킷마다 다를 수 있다. 만약 10개의 패킷이 동시에 비어있는 큐에 도착하면, 첫번째 패킷은 아무런 지연 없이 링크를 통해 전송될 것이다. 하지만 마지막 패킷은 엄청난 Queueing delay 를 하게 된다. 그래서 Queueing delay 를 측정할 때는 평균 지연을 측정한다. 

실제로 지연 과정은 traceroute 라는 프로그램으로 측정할 수 있다. 실제 그 목적지에 도달하기 까지 중간 중간에 있는 라우터들에게 3개의 패킷을 전송해서 반환되는 시간을 측정해준다. 즉 10개의 라우터가 중간에 있으면, 나 -> 라우터 1 -> 나 , 나 -> 라우터2 -> 나  ... 이렇게 패킷의 왕복 시간을 알려준다. 

<img width="588" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3376b73d-0963-4048-a305-eda4b320f618">

### Packet Loss

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/af4a4da4-25cc-42e9-b20e-5c9fea1c7077)

큐는 한정된 용량이 있다. 그래서 라우터의 버퍼가 다 차면, 패킷이 drop 되면서 loss 가 발생한다. 

### Throughput
Throughput 은 송신자와 수신자 사이에 bits 가 전송되는 rate (bits/time) 이다. 만약 Host A 가 전송한 F bits 의 파일을 Host B 가 받는데 T 초가 걸렸다면, 평균 Throughput 는 F/T bits/sec 이다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/82af0ce3-a4e6-415b-8c6b-6ca562890c6c)

두번째 상황처럼 Rs > Rc 일 때, 라우터는 받은 속도만큼 빠르게 패킷들을 전송할 수 없다. 이것이 bottleneck link 이다. 
> bottleneck link : Link on end-end path that constrains end-end throughput

## 1.5 Protocol Layers and Service Models

### Layered Architecture

Layer 는 복잡한 시스템을 다룰 때 좋다. 각 레이어를 모듈화하면, 유지 보수, 업데이트가 쉬워진다. 각 Layer 안에서 생긴 문제는 그 layer 안에서 처리하면 된다. 

하지만 layer 는 완전히 완전히 독립적으로 설계할 수 는 없다. 왜냐면 각 레이어는 다른 레이어에 의존하고 있기 때문이다. 따라서 Layer 설계를 잘 해야 한다.


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Internet Protocol Stack**</span></span>
- application
	- 네트워크 어플리케이션을 지원한다. 
	- FTP, SMTP, HTTP
- transport
	- 데이터 전송을 지원한다.
	- TCP, UDP
- network
	- 소스에서 목적이로의 데이터 라우팅을 지원한다.
	- IP, routing protocols
- link
	- 다른 네트워크끼리의 데이터 전송을 지원한다.
	- Ethernet, 802.111 (Wifi), PPP
- physical
	- bit 로 이루어진 물리적인 신호 

### Encapsulation

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/34576fb7-c9ad-49bc-a558-a750aa4f5c12)

각 레이어를 통과할 때 Header 를 붙인다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 


