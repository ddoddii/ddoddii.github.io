+++
author = "Soeun"
title = "[네트워크] Link Layer and LANs"
date = "2023-11-18"
summary = "Link Layer 에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "network"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "link-layer-and-LANs"
series = ["Network"]
series_order = 15
+++
{{< katex >}}


## 1. Introduction to Link Layer

link-layer 프로토콜에서 작동하는 디바이스(host, router)를 node 라고 하자. 근접한 node 들이 소통하는 통로를 link 라고 하자. datagram 이 발신자 호스트에서 수신자 호스트로 도달할 떄까지, ene-to-end path 내의 개별 링크 를 따라 가야 한다. 링크에는 wired link, wireless link, ethernet link, LAN 등이 있다. 링크 내에서, 전송하는 노드는 datagram 을 link-layer frame 으로 캡슐화 한 다음에, 링크를 통해 전송한다. 

data link layer 는 하나의 노드에서 물리적으로 근접한 다른 노드들에게 datagram 을 전송하는 책임이 있다. 

datagram 은 다른 링크들 내에서 각각 다른 link protocol 에 따라 전송된다. 그리고 각 프로토콜은 다른 서비스들을 제공한다. 

### The services provided by the Link Layer

link layer 에서 제공할만한 서비스들을 보자. 
- Framing 
  
  거의 모든 link layer protocol 은 각 network-layer datagram 을 캡슐화한다. frame 은 data field 들로 구성되는데, 여기에 network-layer datagram 과 다른 헤더 필드들이 있다. 
  
- Link access
  
  MAC(Medium Access Control) protocol 은 frame 이 링크로 전송되는 규칙들을 정의한다. 하나의 발신자와 하나의 수신자만 있는 point-to-point 링크에서는 MAC protocol 이 매우 간단하다. 하지만, 여러개의 노드가 하나의 broadcast link 를 소유하면 상황이 복잡해진다. 여기서 MAC protocol 은 다수의 노드의 frame 전송을 관리한다.
  
- Reliable delivery
  
  Link-layer protocol 이 신뢰할만한 전송 서비스를 제공하면, network layer datagram 이 손실되지 않고 링크를 통해 전송되는 것을 보장한다. TCP 와 비슷하게, 이 서비스는 ack 와 재전송을 통해 구현할 수 있다. 이 서비스는 에러율이 높은 wireless link 에 많이 쓰인다. 하지만 낮은 에러율을 보이는 유선 링크에는 불필요하다. 
  
  
- Error detection and correction 
  
  받는 쪽의 노드는 frame 이 잘못되었는지 알 수 없다. 따라서 다양한 link layer protocol 은 이 bit error 를 감지하는 메카니즘을 제공한다. 

- flow control
  
  발신 노드와 수신 노드 사이에 전송률을 제어할 수 있다. 

### Where is the Link Layer implemented?

Link layer 는 어디에서 구현될까? 이전에 link layer 가 라우터의 line card 에 구현되어 있다는 것을 배웠다. 구체적으로 어떻게 구현되어 있을까?

Link layer 는 **<span style="background:#FEFBD1">network interface card(NIC)</span>** 라고도 불리는 **<span style="background:#FEFBD1">network adapter</span>** 에 구현되어 있다. network adapter 내부에는 link-layer controller 가 있다. 여기서 많은 link-layer service(framing, link access, error detection..) 등이 제공된다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4b6ab633-d2c8-4c71-85ee-845afe455a0f)

2개의 adaptor 가 소통하는 과정을 보자. 
- 보내는 쪽
	- datagram 을 frame 으로 캡슐화한다.
	- error checking bits 를 첨가한다.(rdt, flow control ..)
- 받는 쪽
	- 에러를 확인한다.
	- datagram 을 추출한 후, 받는 쪽의 위쪽 레이어까지 보낸다. 

## 3. Multiple access protocols

2가지 타입의 네트워크 링크가 있다 : point-to-point links, broadcast links. **<span style="background:#FEFBD1">Point-to-point link</span>** 는 양 끝에 수신자와 발신자가 있다. 여기에는 2가지 프로토콜이 있는데, PPP(Point-to-point protocol), HDLC(High-level data link control) 이다. 

**<span style="background:#FEFBD1">Broadcast link</span>** 는 하나의 연결된 채널에 다수의 발신자, 수신자 노드가 연결되어 있다. 수신자가 frame 을 전송하면, 채널은 frame 을 broadcast 하고, 다른 노드들이 모두 복사본을 받는다. 이더넷과 wireless LAN 이 여기에 속한다. 

여기서 가장 중요한 문제는, 다수의 발신자와 수신자 노드가 하나의 broadcast channel 을 공유하는지의 문제이다. 이것을 **<span style="background:#FEFBD1">multiple access problem</span>** 이라고 한다. 만약 다수의 사람들이 동시에 말을 하면, 우리는 제대로 듣지 못할 것이다. 네트워크도 마찬가지이다. 만약 두개 이상의 노드가 동시에 전송을 한다고 하면, **<span style="background:#FEFBD1">interference</span>** 가 발생할 것이다. 그리고 하나의 노드가 2개 이상의 시그널을 동시에 받는다면 **<span style="background:#FEFBD1">collision</span>** 이 발생할 것이다.  따라서 이 문제를 해결하기 위해 **<span style="background:#FEFBD1">multiple access protocols</span>** 가 있다. 

이상적인 multiple access protocol 의 상황을 보자. R bits per second 전송 속도를 가진 broadcast channel 의 예시를 보자. multiple access protocol는 다음과 같은 특성들을 가져야 한다.
1. 하나의 노드만 전송할 데이터를 갖고 있다면, 그 노드는 R 의 속도로 전송할 수 있다. 
2. M 개의 노드들이 전송할 데이터가 있다면, 각 노드는 R/M 의 속도로 전송할 수 있다. 
3. 프로토콜은 decentralized 이다 ; 즉, 모든것을 제어하는 하나의 마스터 노드는 없다.
4. 프로토콜은 단순하고, 구현하기 쉽다. 

### Channel Partitioning Protocols

앞서서, 채널의 bandwidth 를 나누는 TDM, FDM 에 대해서 배웠다. 여기서도 비슷하다. 

#### TDMA(Time Division Multiple Access)
N 개의 노드가 있고, 전송률이 Rbps 이라고 하자. TDMA 에서는 시간을 N 개의 time slot 으로 나눈다. 각 station 은 라운드 마다 fixed length slot을 할당받는데, 보통 length 는 하나의 패킷을 전송하는데 걸리는 시간으로 정한다. TDMA 는 완벽하게 공정하다.  각 노드는 라운드마다 R/N 의 전송률을 보장 받는다. 

하지만 이때 사용되지 않는 slot 은 idle 하다는 단점이 있다. 또, 만약 하나의 노드만 전송을 할 때도 전송률이 R/N 으로 제한된다는 단점이 있다. 

아래 예시는, 6-station LAN 에서 1,3,4 는 전송할 패킷이 있지만 2,5,6 은 전송할 패킷이 없는 상태이다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cdaac495-ca74-4de1-b887-f1140e5809a5">

#### FDMA(Frequency Division Multiple Access)
채널의 주파수를 나누는  방식이다. 각 station 은 고정된 frequency band 가 할당된다. 하지만 여기서도 사용되지 않는 주파수는 idle 하다는 문제점이 있다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/957b39e2-55ea-407b-ac96-7cd07a89b8d4">

#### CDMA(Code Division Multiple Access)
TDM 와 FDM 은 time slots / frequency 를 할당하지만, CDMA 는 각 노드에게 code 를 할당한다. 각 노드는 유니크한 코드를 이용하여 데이터를 전송한다. 더 자세한 것은 ch.7 에서 배울 것이다.

### Random Access Protocols

노드가 전송할 패킷이 있을 때, 채널의 data rate 인 R 로 패킷을 전송한다. 노드간에 사전 합의는 없다. 만약에, 2개 이상의 전송하는 노드가 있는 경우 collision 이 발생할 수 있다. collision 이 발생하면, 패킷을 바로 재전송하는 대신 각 노드들은 random delay 만큼 기다렸다가 재전송한다. 

세부적인 프로토콜마다, 어떻게 collision 을 감지하고, 이것을 어떻게 해결하는지는 다르다. 

Random access protocol 의 예시로는 , slotted ALOHA, ALOHA,CSMA, CSMA/CD, CSMA/CA 가 있다. 

#### Slotted ALOHA

Slotted ALOHA 는 아래의 사항들을 가정한다. 
- 모든 frame 은 같은 사이즈이다.
- 시간은 L/R sec 의 slot 으로 나눠진다. (slot 은 하나의 frame 을 전송하는데 걸리는 시간과 동일하다.)
- 노드들은 slot 의 시작에만 frame 을 전송하기 시작한다.
- 노드들은 모두 동기화되어 있고, 노드는 slot 이 언제 시작하는지 알 고 있다. 
- slot 내에서 2개 이상의 노드가 충돌하면, 모든 노드들이 충돌 소식을 알 수 있다. 

어떻게 작동하는지 알아보자.
- 노드가 보내야할 fresh frame 이 있을 때, 다음 slot 의 시작까지 기다렸다가 slot 안에서 entire frame을 전송한다. 
- 충돌이 없는 경우에, 노드는 frame 을 성공적으로 전송했으며, 재전송하지 않는다.
- 충돌이 발생하면, 노드는 slot 이 끝나기 전에 충돌을 감지한다. 노드는 후에 오는 slot 마다 성공할 때까지 확률 p 로 frame 을 재전송한다. 

p 의 확률로 전송하다는 것은, p 는 "재전송" 을 의미하고, 1-p 는 그 slot 을 스킵한다는 확률이다. 

slotted ALOHA 는 항상 노드가 패킷을 full rate R 로 전송할 수 있게 해준다. 또, 매우 decentralized 되어 있고, 간단하다. 하지만 단점으로는 collision 이 있고, 쓰이지 않는 slot 들이 있다는 것이다. 또, 노드간의 동기화가 필요하다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/70844761-51cf-427c-ba4d-92c8c1779d23">

만약 충돌이 일어난다면, 그 slot 은 "wasted"된 것이다. 또, 패킷 전송이 일어나지 않아서 빈 slot 도 "wasted" 된 것이다. 그렇다면 "unwasted" 된 slot 은 하나의 노드만 패킷을 전송할 때이다. 이것을 successful slot 이라고 한다. **<span style="background:#FEFBD1">Efficiency</span>** 는 장기적으로 봤을 때 많은 노드들이 많은 frame 을 전송하는 상황에서 successful slot 의 비율이다. 

Slotted ALOHA 의 efficiency를 계산해보자. N 개의 노드가 보내야할 많은 frame 들이 있고, 각 노드는 확률 p 로 slot 에 전송한다. 하나의 노드가 slot 에서 success 할 확률은 \\(p(1-p)^{N-1}\\) 이다. 즉, 특정 노드만 보내고 나머지 노드들은 전송하지 않는 경우이다. Any 노드가 성공할 확률은 \\(Np(1-p)^{N-1}\\)  이다.

여기서 max efficiency 는 , \\(Np(1-p)^{N-1}\\) 를 최대화하는 \\(p^*\\) 을 구하면 된다.  \\(Np(1-p)^{N-1}\\) 의 리밋을 구하면, max efficiency 는 \\(1/e=3.7\\) 이 된다. 즉, 가장 이상적일 때, 채널은 37% 의 확률로 전송에 성공한다. 


#### Pure (unslotted) ALOHA 

unslotted ALOHA 는 synchonization 기능이 없다. 즉, frame 을 slot 의 시작시간에 맞추서 전송하는 것이 아니라 frame 이 도착하면 바로 전송한다. 하지만 이때 충돌 확률이 증가한다. \\(t_{0}\\) 에 보내진 frame 은 \\([t_{0}-1,t_{0}+1]\\)  시간에 보내진 frame 과 충돌한다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cfc0be22-252c-4de8-b634-4d8583253656">

Pure ALOHA 의 efficiency 를 계산해보자. 

$$P(success\ by \ given \ node) =  P(node\ transmits)*P(no\ other\ node\ transmits \ in [t_{0}-1,t_{0}])*P(no\ other\ node\ transmits \ in [t_{0},t_{0}+1])$$

$$=p*(1-p)^{N-1}*(1-p)^{N-1}$$

$$=p*(1-p)^{2(N-1)}$$

n 을 리미트로 보내고, 최적화된 p 를 고르면, max. effiency 는 0.18 이 나온다. 즉, clock synchro 에 들어가는 overhead 는 없어졌지만, 효율을 오히려 slotted ALOHA 보다 나빠졌다 !

#### CSMA(Carrier Sense Multiple Access)

ALOHA 에서는 패킷을 전송하는 것은 노드의 결정이었다. 즉, 다른 노드들이 전송하고 있건 말건 신경쓰지 않는다. CSMA 는 전송하기 전에 "listen" 하는 단계가 있다. 이것을 **<span style="background:#FEFBD1">carrier sensing</span>** 이라고 한다. 만약 채널이 idle 하다면, 전체 패킷 을 전송한다. 전송하는 와중에도, 다른 노드가 패킷을 전송하는 것을 감지한다면 (**<span style="background:#FEFBD1">collision detection</span>**) 전송하는 것을 멈춘다. 

모든 노드들이 carrier sensing 을 하는데, 어떻게 충돌이 일어나는가? 아래의 다이어그램에서 볼 수 있다. \\(t_0\\) 에 B 는 채널이 idle 하다는 것을 알고, 전송을 시작한다. \\(t_1\\) 에, D 는 전송할 패킷이 생긴다. B 는 이때 여전히 전송중이지만, B 가 전송한 패킷의 bit 가 아직 D까지 도착하지 않았다. 따라서 D는 \\(t_1\\) 에 채널이 idle 하다고 느낀다. 따라서 D 는 패킷을 전송하기로 시작한다. 조금 후에, B 는 D의 전송과 interfere 하기 시작한다. 따라서, end-to-end **<span style="background:#FEFBD1">channel propagation delay</span>**(하나의 노드에서 다른 노드들로 propagate 하는데 걸리는 시간) 가 성능에 굉장히 중요한 역할을 한다는 것을 알 수 있다. 

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bb969a2d-6040-4e0b-8c17-348b15654bc0">

#### CSMA/CD(Carrier Sense Multiple Access with Collision Detection)

위에 CSMA 에서는 , 노드가 collision detection 은 하지 않았다. B,D 는 충돌이 일어난 후에도 본인의 패킷을 계속 전송한다. collision detection 은 , 충돌을 감지하면 전송을 멈추는 것이다. 

아래는, 위의 시나리오와 동일하지만, 충돌을 감지한 후에 전송을 짧은 시간 동안 멈췄다. Collision detection 을 추가하는 것은 손상된 패킷을 그대로 전송하는 것을 방지해준다. 

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e2d3d7fe-be18-4f52-b75d-2a26686866d7">

broadcast channel 에 붙어있는 노드의 입장에서 살펴보자.
1. 노드가 network layer 에서 datagram 을 받고, link-layer frame 을 준비한 후에, 이 frame 을 buffer 에 넣는다. 
2. 노드는 채널이 idle 하다는 것을 감지하고, frame 전송을 시작한다. 만약에 채널이 바쁘다는 것을 감지하면, 다른 시그널이 없을 때까지 기댜렸다가 frame 을 전송한다. 
3. 전송 중에, 노드는 broadcast channel 을 사용하여 들어오는 다른 시그널이 없는지 감지한다. 
4. 노드는 다른 시그널이 없이 frame 전체를 전송하면 끝난다. 하지만, 전송 중에 다른 시그널을 감지하면, 전송을 중지한다. 
5. 중지 후에 , 노드는 random 한 시간을 기다린 다음에 step 2 로 돌아간다. 

랜덤한 시간을 기다려야 하지만, 대략 어느 구간에서 골라야 할까? 구간이 짧고 전송하는 노드들이 많으면 같은 시간을 고를 확률이 높아져 계속 충돌할 것이고, 구간이 길고 전송하는 노드들이 적다면 너무 많은 시간을 기다려야 할 것이다. 따라서, 최적의 구간은 노드들이 적을 때는 구간의 길이가 짧고, 충돌하는 노드들이 많으면 구간의 길이가 길 때이다. 

**<span style="background:#FEFBD1">Binary exponential backoff</span>** 알고리즘은 이더넷에 의해 사용되는데, 이 문제를 해결해준다. 이미 n번의 collisions 를 경험한 frame 을 전송할 때는, \\(\{0,1,2,...,2^n-1\}\\) 에서 랜덤한 값 K 를 선택한다. 이렇게 하면 충돌한 횟수가 커질수록 선택하는 구간의 폭도 커진다. 

그렇다면, CSMA/CD 의 efficiency 를 보자. 
\\(T_{prop}\\) = max  propagation  delay  between  2  nodes in LAN
\\(t_{trans}\\) = time to transmit max-size frame

\\(efficiency= \frac{1}{1+5t_{prop}/t_{trans}}\\)  

\\(t_{prop}\\) 가 0 으로 수렴하고, \\(t_{trans}\\) 가 무한대로 가면, efficiency 는 1로 수렴한다. 즉, ALOHA 보다 더 좋은 성능을 낸다. 

### Taking-Turns Protocols

앞에서, multiple access protocol 의 이상적인 조건들은 다음과 같았다.  (1) 하나의 노드만 전송하면, R 의 속도로 전송한다. (2) M 노드들이 active 하다면 각 노드들은 R/M bps 의 속도로 전송한다. 앞에서 본 CSMA, ALOHA 는 첫번째 조건은 만족하지만 두번째 조건은 만족하지 않는다. 따라서 새로운 프로토콜인 **<span style="background:#FEFBD1">Taking-Turns protocol</span>** 이 나왔다.  구체적으로 구현하는 프로토콜은 여러가지이지만, 여기서는 먼저 polling protocol 에 대해 알아보자. 

#### Polling Protocol

여기서는 하나의 노드가 마스트 노드로 지정된다. 마스터 노드는 slave 노드에게 전송하라고 초대(poll)한다. 예를 들어, 마스터 노드는 1번 노드에게 메세지를 보내서 frame 을 전송하라고 말한다. 그 후 다른 노드들에게도 이 작업을 반복한다. 마스터 노드는 노드가 frame 전송을 끝냈는지 채널의 시그널을 관찰해서 알 수 있다. 

Polling protocol 은 충돌과 빈 slot 이 있는 것을 방지한다. 하지만 단점도 있다. 첫번째는 , polling delay 가 있다는 점이다. 예를 들어 하나의 노드만 active 하다면, 그 노드는 R bps 보다 작은 속도로 전송한다. 왜냐하면 마스터 노드는 각각 inactive 노드들에게 active 노드가 frame 전송이 끝난 후 메세지를 보내야 하기 때문이다. 두번째는, 마스터 노드가 fail 하면 전체 프로토콜이 fail 한다. 

<img width="300" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7db6edeb-6286-4284-a398-9517a1cb7989">

#### Token-passing protocol
여기서는 마스터 노드가 없다. 대신, 노드끼리 **토큰**을 전달한다. 이때 전달은 고정된 순서를 따른다. 만약 노드가 토큰을 건내 받으면, 전송한 frame 이 있을 때만 토큰을 가지고 있는다. 만약 frame 이 없다면 다른 노드에게 토큰을 전달한다. 

하지만 단점이 있다. token overhead 가 발생할 수 있고, 토큰이 사라지거나 하나의 노드가 망가지면 전체가 무너질 수 있다. 

<img width="300" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d6e7da65-dbb5-4fb1-8d14-78df44f2d8ee">


### DOCSIS : The Link-Layer Protocol for Cable Internet Access

Cable internet 에서 위에서 알아본 프로토콜이 어떻게 작동하는지 알아보자. Cable network 는 수천개의 가정에서 cable modem 들을 연결하여 CMTS(cable modem termination system) 에 연결한다. DOCSIS(Data-Over-Cable Service Interface Specifications) 는 이것을 결정한다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/66259853-3351-41bf-922d-2d0fb5928373">


## 4. Switched Local Area Networks(LANs)
이제 local switches 에 대해 알아보자. 아래는 3개의 department 를 연결하는 local switch 를 보여준다. 이 스위치는 2개의 서버와 4개의 스위치가 있는 라우터를 연결한다. 이 스위치들은 link-layer 에서 작동하기 때문에 netword-layer 주소인 IP 주소를 사용하지 않는다. 

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/60825475-5875-4e4a-9da2-4ffa4cbe741a">

### Link-Layer Addressing and ARP

대신, link-layer 에서 사용되는 주소가 있다. 이미 앞에서 network layer 에서 사용하는 IP 주소가 있는데 왜 또 주소가 필요할까? 라고 생각할 수 있다. 차차 알아보자. 

#### MAC Address

실제로는, 호스트와 라우터가 link-layer 주소를 가지는 것이 아니라, 그들의 adapter(network interface) 가 link-layer 주소를 가지는 것이다. 만약에 호스트가 다수의 adapter 가 있다면 여러개의 link-layer 주소를 가질 수 있다. 

여기서 사용되는 주소는 **<span style="background:#FEFBD1">Interface address (MAC/LAN/Ethernet/physical/hardware address)</span>** 라고 불린다. 이것은 같은 physical layer network 내에 있는 디바이스 끼리 data frame 을 국지적으로 전송하는데 사용된다. (Ethernet, Wifi...)

MAC 주소는 NIC ROM 에 각인되어 있지만, 소프트웨어로도 세팅할 수 있다. 예를 들어 이더넷은 48bits(= 6 bytes)로, 6 octets 으로 나타낸다 : 1a-2f-bb-76-09-ad. 어느 2개의 adapter 도 같은 MAC 주소를 가지지 않는다. 

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5ba45149-67bb-453b-bea4-cd55ca6dffde">

IEEE 가 MAC 주소를 관리한다. NIC 제조사는 MAC 주소의 부분을 산다. 첫번째 24 bits 는 제조사를 나타낸다. MAC 주소는 flat structure 이고, 장소를 이동하던 절대 바뀌지 않는다. IP 주소와 비교하자면 , IP 주소는 hierarchical structure 이었고, 장소를 이동하면 subnet 이 바뀌면서 변경되었다. 비유하자면 MAC 주소는 주민번호와 같고, IP 주소는 집 주소와 비슷하다. 

하나의 adapter 가 다른 adapter 에게 frame 을 보내고 싶다고 했을때, 발신자는 수신자의 MAC 주소를 frame 안에 넣고 LAN 에 frame 을 전송한다. 스위치는 들어오는 frame 을 모든 interface 에게 broadcast 한다. frame을 받은 adapter 는 자신에게 온 frame 인지 확인하고, 맞으면 datagram 을 꺼내서 protocol stack 에 저장하고, 아니면 버린다. 

하지만 만약에 발신 adapter 가 LAN에 있는 모든 adapter 가 frame 을 받기를 원하면 어떻게 해야 할까? 이런 경우에는 MAC broadcast address 를 넣는다. 이 주소는 연속된 48개의 1 로 구성되어 있는데, hexadecimal notation 으로 표기하면 **FF-FF-FF-FF-FF** 이다. 

#### Address Resolution Protocol(ARP)

IP 주소와 MAC 주소 2개가 있는데, 그러면 두개의 주소를 어떻게 서로 번역할까? 이때 쓰이는 것이 **<span style="background:#FEFBD1">Address Resolution Protocol(ARP)</span>** 이다. 

아래 처럼, 각 호스트는 1개의 IP 주소와 1개의 MAC 주소를 가지고 있다고 해보자. 그리고, LAN 내에 있는 스위치가 frame 을 받으면 다른 모든 interface 에게 이 frame을 전송한다고 가정하자.

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/53656f31-68e8-4b59-81f3-c6c27ed0affe">

제일 위에 있는 호스트가 아래에 있는 호스트에게 frame 을 전송하고 싶다고 하자. 여기서 source 와 destination 모두 같은 subnet 에 있다. datagram 을 전송하기 위해서는 발신자는 수신자의  IP주소 뿐만 아니라 MAC 주소도 알아야 한다. 그렇다면, IP 주소는 DNS 서버로 부터 받을 수 있다는 것은 아는데, 이 IP 주소를 가지고 어떻게 MAC 주소를 알 수 있을까?

정답은 ARP 이다. 호스트에 있는 IP node 는 **<span style="background:#FEFBD1">ARP table</span>** 이 있는데, 같은 LAN에 있는 IP 주소를 입력값으로 받으면 그에 할당하는 MAC 주소를 돌려준다. DNS 와 비교하자면, DNS 는 인터넷 어디에 있든 호스트 이름을 통해 IP 주소를 돌려주지만, ARP 는 같은 subnet 에 있는 호스트의 IP 주소만 다룬다. 

ARP Table 는 아래와 같이 <IP address, MAC address, TTL> 을 포함한다. TTL(Time To Live) 는 주소 매핑이 삭제되기 까지의 시간이다. 

<img width="450" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fad87566-2a0e-4190-8cc0-04d3afad2d26">

그렇다면 이제 ARP Table 이 어떻게 작동하는지 보자. 각 호스트와 라우터는 메모리 안에 ARP Table 이 있다. 발신자의 (IP 주소 - MAC 주소) 쌍이 이미 ARP Table 에 있다면 일은 아주 쉬워진다. 

그러나 이 쌍이 없다면 어떻게 할까? A 가 B 로 보내는 상황을 보자. 첫번쨰로 A는 **<span style="background:#FEFBD1">ARP query packet</span>** 이라는 특별한 패킷을 준비한다. ARP packet 은 여러가지 특별한 필드가 있는데, 여기에는 B의 IP주소가 포함되어 있다. 그 후 이 ARP Packet 을 LAN 전체에 **broadcast**한다.  이때 목적지 MAC 주소는 FF-FF-FF-FF-FF-FF 로 표기한다. 그러면 LAN 에 있는 모든 노드가 이 패킷을 받는다. 모든 노드는 이 패킷을 받으면 본인의 IP주소가 패킷에 적힌 IP주소와 일치하는지 확인한다. B가 ARP 패킷을 받으면, A에게 본인의 MAC 주소를 적어서 답장한다(**unicast**). A는 (IP-MAC) 쌍을 ARP Table 에 캐싱한다.  ARP 는 '**plug-and-play**' 인데, ARP table 은 관리자의 승인 없이 자동으로 만들어진다. 

여기서 ARP 는 link-layer 프로토콜인지, network-layer 프로토콜인지 헷갈릴 수 있다. ARP 는 link-layer frame 내에 캡슐화 되므로, 엄밀히 link-layer 위에 있다. 하지만, ARP 는 link-layer주소와 관련된 필드가 있다. 또, network-layer 주소와 관련된 필드도 있다. 따라서 그냥 link-layer, network-layer 사이에 있는 프로토콜로 정의한다. 

ARP 메세지는 link-layer frame 의 data portion 에 들어간다. 
- ARP query
	- destination address : FF-FF-FF-FF-FF-FF
	- source address : 디바이스 본인의 MAC address
- reply
	- destination address : ARP Query 를 보낸 디바이스의 MAC address
	- source address : ARP Query 를 응답하는 디바이스의 MAC address

ARP 메세지가 min. data link frame 보다 작으면 data portion 은  0 으로 패딩된다. 이더넷에서는, 0x0806 이라는 frame type field 가 사용된다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**ARP Packet Format**</span></span>  

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c90357cc-d9e0-4759-ab55-a61b934af678">

<span style="font-size:110%"><span style="background-color: #EBFFDA">**ARP Cache**</span></span>  

각 IP datagrame 마다 ARP request/reply 를 보내는 것은 비효율적이므로, 호스트는 최근 엔트리를 캐싱한다. ARP 캐시의 엔트리는 expiration time 이 있다. 이 유효시간이 끝날때까지 엔트리는 업데이트 되지 않는다.

아래는 **ARP Cache Table** 이다. 

<img width="400" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fe9304f2-0dd3-4e67-ab56-4c4731b10e62">

#### Sending Datagram off the subnet

그럼 호스트가 다른 LAN에 있는 호스트에게 datagram 을 어떻게 보낼까? 아래의 예시를 보자. subnet 1 에 속하는 네트워크는 모두 111.111.111.xxx 의 IP주소를 가지고, subnet 2 에 속하는 네트워크는 모두 주소 222.222.222.xxx 의 IP주소를 가진다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1200217b-de22-4bce-aba7-0fcb9e50f174">

A 가 B에게 datagram 을 보낸다고 하자. 우선, A 는 목적지로 가는 path 중 first-hop 라우터인 R에게 datagram 을 보낸다.  A 는 R의 **IP 주소는 DHCP 서버를 이용**해서, **MAC 주소를 ARP 를 사용**해서 알아낸다. 이 datagram 은 IP datagram(A의 IP주소, B의 IP주소) 가 포함되어 있다. A는 그 후 link-layer frame에 R의 MAC 주소를 목적지 주소로 적는다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/500d92a7-f80d-4012-aada-43562f967391">

이 frame 은 A에서 R 로 보내진다. R이 frame 을 받으면, datagram 을 추출해서 IP layer 로 보낸다. 그 후 R 는 (IP souce A, destination B) 로 된 datagram 을 포워딩한다. link-layer frame 에는 B의 MAC 주소가 목적지 주소로 적는다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/672b1124-a01e-44c5-863f-e833dab0856c">

그럼 이 datagram 이 B에 도착한다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ca42c3f2-a988-4a65-860f-84d2dc0245cb">

#### Proxy ARP

만약에 호스트가 본인의 네트워크를 더 큰 범위로 보게 된다면 어떻게 할까? 더 큰 범위로 인식한다는 것은, prefix 가 더 짧아진다는 것이다 호스트 223.1.1.1/16은 223.1.xxx.xxx 를 가진 PC를 모두 본인의 로컬 서브넷에 있다고 판단하고, 라우터에게 물어보지 않고 바로 ARP 를 보낼 것이다. 

더 풀어쓰자면, 호스트 223.1.1.1/16 의 네트워크 뷰나, 호스트 223.1.3.1/16 모두 223.1.0.0 일 것이다. (뒤에 16이 있으므로 앞에서부터 16bit 를 고정한다. 즉 앞에 16bit 가 같으면 로컬 서브넷으로 인식한다는 의미이다.)

같은 서브넷에 있다고 인식하면,  (호스트) 223.1.1.1/16 는 (PC2) 223.1.3.1/24 에게 바로 ARP 를 보낼 것이다. 하지만 PC2 는 이것을 응답할 수 있을까? 아니다. 둘은 실질적으로 같은 네트워크 안에 있지 않으므로 ARP 에 대해 응답하지 않을 것이다. 이것을 위한 해결책이 **<span style="background:#FEFBD1">Proxy ARP</span>** 이다. 

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f2beb1e0-579b-4d64-b398-fc124e0f5a06">

호스트는 PC2 에게 바로 ARP를 보내지만, 라우터가 개입한다. 라우터는 PC2 인척 하면서 , 호스트에게 본인의(라우터) MAC 주소를 보낸다. 이것을 **<span style="background:#FEFBD1">proxy ARP</span>** 라고 한다. 이것은 라우터가 호스트가 PC2 와 둘 다 **직접적**으로 연결되어 있을 때만 작동한다. 

두가지 ARP 상황에서 ARP 캐시를 보자.
- Non Proxy ARP - Host /24:
	- 호스트는 라우터에게 ARP 를 보낸다. (정상적으로 작동)
	- 호스트의 ARP 캐시는 라우터의 IP주소를 가진다. 
	- (라우터의 IP주소-라우터의 MAC주소)
- Proxy ARP - Host /16:
	- 호스트는 PC2 에게 ARP 를 보내려고 할 것이다. 이때 라우터는 proxy ARP 를 사용한다. 
	- 호스트의 ARP 캐시는 PC2의 IP 주소를 가진다.
	- (PC2의 IP주소 - 라우터의 MAC주소)

### Ethernet

이더넷은 LAN 기술에서 독보적이다. 가장 처음으로 사용된 LAN 기술이었고, 간단하고 하드웨어 싸다. 

이더넷의 물리적인 구조를 보자면 90년대까지는 bus 가 유행했다. 모든 노드가 같은 collision domain 에 있다. 그리고 CSMA/CD 와 shared bus 를 이용한다. 전송 속도는 약 10Mb/s 이었다. 

그 이후에는 , hub-based star 기술으로 교체되었다. 허브는 물리적 레이어의 디바이스로, frame 말고 bits를 기반으로 작동한다. 2000년대에는 또 큰 변화를 겪었는데, 허브 대신 switch 를 사용하기로 했다. 여기서 노드들은 다른 노드들과 충돌하지 않는다. 그리고 전송 속도도 100Mb/s , Gigabit 이더넷은 1Gbps 로 훨씬 빨라졌다. 

<img width="401" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3ff92026-843f-46cd-a1de-84378e241aa8">

#### Ethernet Frame Structure

이더넷 프레임에 대해 알아보기 위해 호스트A 가 같은 LAN에 있는 다른 호스트B 에게 IP datagram 을 전송하는 상황을 보자. 보내는 쪽의 adapter 는 AA-AA-AA-AA-AA-AA 의 MAC 주소를 가지고, 받는 쪽의 adapter 는 BB-BB-BB-BB-BB-BB 의 MAC 주소를 가진다. 보내는 쪽의 adapter 는 IP datagram 을 이더넷 프레임 안에 캡슐화하고, 이 frame 을 물리적 레이어로 보낸다. 받는 쪽의 adapter 는 frame 을 물리적 레이어에서 받아서, IP datagram 을 꺼내서 네트워크 레이어로 보낸다. 

여기서 이더넷 프레임의 6가지 필드를 보자. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0326d6c4-289a-4492-bb96-9db7675d384c">

- Data field(46 - 1,500 bytes)
  
  이 필드는 IP datagram 을 포함한다. 이더넷의 MTU(Max. Transmission Unit) 은 1,500 byte 이다. 이것은 만약 IP datagram 이 1,500 byte를 초과하면 호스트는 datagram 을 쪼개서 보내야 한다. 만약 46 byte 보다 작다면 호스트는 46byte 가 될때까지 채워야 한다. 
  
- Destination address(6 bytes)
  
  목적지 adapter 의 MAC 주소이다. adapter B 가 BB-BB-BB-BB-BB-BB 의 MAC 주소가 적힌 이더넷 프레임을 받으면, 이더넷 프레임의 data field 를 네트워크 레이어로 넘긴다. 만약 다른 MAC 주소가 적혀있으면 프레임을 버린다.
  
- Source address(6 bytes)
  
  보내는 쪽의 MAC 주소가 적혀있다. 
  
- Type fields(2 bytes)
  
  이 필드는 이더넷이 여러가지 네트워크 레이어 프로토콜을 쓸 수 있다는 것을 의미한다. 주로 IP 프로토콜이지만, 다른 프로토콜을 나타낼 수 있다. 
  
- CRC(Cyclic Redundancy Check, 4 bytes)
  
  CRC 는 받는 쪽의 adapter 가 프레임 내부의 에러를 체크할 수 있게 한다. 
  
  
- Preamble(8 bytes)
  
  이더넷 프레임은 8 byte 의 preamble 필드로 시작한다. 첫 7byte 는 모두 10101010 으로 되어 있고, 마지막 byte 는 10101011 이다. 이것은 받는 쪽의 adapter 를 "깨우기" 위해서이다. 보내는 쪽은 프레임을 10Mbps 또는100Mbps 또는 1 Gbps 로 보내길 희망하지만, 언제나 완벽할 수는 없다. 따라서 받는 쪽은 보내는 쪽의 clock 와 synchronize 하는 작업이 필요한데, 이 필드가 그 역할을 한다. 마지막 byte 의 연속된 '11' 은 '이제 큰거 온다 !!' 라는 뜻을 가지고 있다. (ㅋㅋ)

이더넷는 unreliable, connectionless 이다. 
- connectionless 
  
  받는쪽과 보내는 쪽의 NIC 사이에 handshaking 과정이 없다.
  
- unreliable 
  
  받는 NIC 는 보내는 NIC 에게 ACK, NACK 를 보내지 않는다. 받는 쪽은 단순히 CRC 를 확인해서 에러가 있는지 체크한다. 프레임이 CRC 체크를 통과하지 못하면, 그 프레임을 버린다. 프레임이 버려져도 보내는 쪽은 이게 버려졌는지 여부를 알지 못한다. 이것은 네트워크 레이어로 전송된 datagram 이 gap 을 가질 수 도 있다는 의미이다. 
  
  만약 이더넷 프레임이 버려져서 gap 이 생긴다면, 받는 쪽의 어플리케이션도 gap 을 알 수 있을까? 이것은 어플리케이션이 TCP / UDP 를 사용하는지에 따라 달려있다. 어플리케이션이 UDP 를 사용하면, 받는 쪽은 gap 을 알아차릴 것이다. 만약 TCP 를 사용하면, 받는 쪽의 TCP 는 ACK 를 보내지 않을 것이고, 보내는 쪽은 재전송을 할 것이다. 여기서 이더넷도 재전송을 하지만, 이더넷은 이게 재전송을 하는 것인지 새로운 datagram 을 보내는 것인지는 알지 못한다. 

#### Ethernet Technologies

이더넷을 실제로 구현하는 기술은 정말 다양하다. 하지만 가장 표준화된 것은 IEEE 의 802.3 CSMA/CD 기술이다. 

다양한 이더넷 표준이 있다. 공통적인 MAC 프로콜과 frame format 이 있고, 다양한 속도 (2Mps, 10Mbps, 100Mbps, 1Gbps, 10Gbps, 40Gps...) 이 있다. 물리적 레이어 미디어 에도 fiber, copper 등이 있다. 

  <img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/24052a90-663a-4192-89a1-edae8dbaa657">
  
  100BaseT 의 이더넷을 보자. 첫번째 100은 속도를 의미한다. BASE 는 이더넷 baseband 를 의미한다. 마지막 부분은 물리적 미디어를 뜻하는데 T 는 twisted-pair copper wire 이다. 

### Link-Layer Switches

스위치의 역할은 들어오는 link-layer frame 들을 outgoing link 로 전송하는 것이다. 스위치 자체는 사실 존재감이 없다. 무슨 뜻이냐면 프레임을 보내는 호스트/라우터는 스위치의 존재를 모른채 프레임을 LAN 에 보낸다. 스위치에게 도착하는 프레임의 속도가 보내는 속도보다 많을 때를 대비에서 스위치 인터페이스도 buffer 가 있다. 


#### Forwarding and Filtering

**Filtering** 은 프레임이 다른 interface 로 보내질지 혹은 버릴 지 결정하는 스위치의 기능이다. **Forwarding** 은 어느 인터페이스로 보낼지 결정하고, 프레임을 보는 기능이다. 이 과정은 **switch table** 을 이용한다. 

Switch table 은 LAN 에 있는 몇몇 엔트리들을 포함한다. 이 엔트리에는 (1) MAC 주소, (2) 그 MAC 주소로 보내는 스위치의 인터페이스, (3)
 엔트리가 테이블에 적힌 시간 이 있다. Switch table 은 MAC 주소를 기반으로 프레임을 forwarding 한다. Switch table 은 router 의 forwarding table 과는 매우 다르다. 

목적지 주소 DD-DD-DD-DD-DD-DD 가 적인 프레임이 스위치의 인터페이스 x에 도착한다고 하자. 스위치는 이것을 테이블에 인덱싱한다. 이 때 3가지 시나리오가 있다.

- 테이블에 DD-DD-DD-DD-DD-DD 의 엔트리가 없다. 이 경우에는 스위치가 프레임의 복사본들을  x 를 제외한 모든 인터페이스의 output buffer 에 포워딩한다. 다른 말로, 스위치가 프레임을 브로드캐스팅한다. -> **flood**
- 엔트리가 있는데, (DD-DD-DD-DD-DD-DD - x) 가 연결되어 있다. 이 경우에는 이 프레임은 다른 인터페이스에 포워딩할 필요가 없으므로, 스위치는 프레임을 버린다. 
- 엔트리가 있는데, (DD-DD-DD-DD-DD-DD - \\(y \not = x\\() 이다. 이 경우에는 프레임은 인터페이스 y 로 포워딩되어야 한다. 이 때 스위치는 인터페이스 y 에 있는 output buffer 로 포워딩 한다.

이것을 pseudo code 로 표현하면 아래와 같다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f73577da-5c58-4143-8708-894c466efc6d">


여기까지 봤을 때 스위치는 hub 보다 매우 똑똑해 보인다. 그런데 애초에 스위치는 어떻게 구성되는 것일까? 모든것을 통제하는 매니저가 있는 것일까? 아니다. 스위치는 self-learning 이라는 훌륭한 기술이 있다.

#### Self-Learning

스위치의 테이블은, 자동적으로 , 다이나믹하게, 동시에 세팅된다. 이것을 **<span style="background:#FEFBD1">self-learning</span>** 이라고 한다. 

1. Switch table  은 처음에 비어있다.
2. 인터페이스에 들어오는 frame 마다, 스위치는 테이블에 (1) frame source 의 MAC 주소, (2) frame 이 도착한 인터페이스, (3) 현재 시간 을 적는다. 
3. aging time 동안 그 source 주소로 부터 다른 frame 이 도착하지 않으면 스위치는 테이블에 있는 주소를 지운다. 

스위치는 **'plug-and-play'** 이다. 왜냐면 네트워크 관리자가 전혀 개입하지 않기 때문이다. 스위치는 **full-duplex** 한데, 이것은 스위치 인터페이스가 동시에 보내고 받을 수 있기 때문이다. 

Self-learning switch 는 같이 연결될 수 도 있다. (multi-switch)

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/795638f6-3ac0-434a-a89e-64391fc58ae3">

A -> G 까지 프레임을 보내고자 한다면, S1 은 프레임을 S4, S3 를 통해서 보내야 하는 것을 어떻게 알까? 이것도 self-learning 으로 해결할 수 있다. 먼저 C가 I에게 프레임을 보내고, I 가 응답했다고 하자. 이때 각 스위치 테이블에 있는 정보는 아래와 같다. 이렇게 S1 는 S3 까지 가려면 4번 포트를 이용해야 하고, S4 는 S3 으로 가려면 3번 포트를 이용해야 한다는 사실을 테이블에 적어놓게 된다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1138a677-47bd-49d9-9928-c64af75b5d3c">

#### Properties of Link-Layer Switching

스위치에 대해 더 깊이 알아보자. bus, hub 대신 스위치를 사용하는 여러 장점들에 대해 알 수 있다. 
- Elimination of collisions
  
  스위치로부터 만들어진 LAN 은, 충돌로 인해 버리는 bandwidth 가 없다. 스위치는 프레임을 버퍼에 저장하고, 하나의 segment 에 동시에 2개의 프레임을 보내지 않는다. 
- Heterogeneous links
  
  스위치가  링크들을 분리하기 때문에, LAN 에 있는 링크들은 다른 미디어, 다른 속도로 작동할 수 있다. 
  
- Management
  
  만약 adapter 가 고장나면, 스위치는 이 고장난 adapter 를 연결을 끊을 수 있다. 

#### Switches vs. Routers

앞에서 라우터는 네트워크 레이어 주소를 이용해서 패킷을 store-and-forward 한다. 스위치도 비슷하게 store-and-forward 를 하지만, 스위치는 MAC 주소를 사용한다. 

<img width="300" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fb18d845-df45-4b73-b5a1-a24738ad1db0">

위에서 볼 수 있듯이, 데이터를 보낼 때 스위치 대신 라우터를 선택할 수 도 있다. 그렇다면 두가지의 차이점은 무엇일까?

|                   | 스위치                                  | 라우터               |
| ----------------- | --------------------------------------- | -------------------- |
| layer             | link-layer device                       | network-layer device |
| forwarding tables | flooding, self-learnng, MAC 주소를 이용 | routing algorithm, IP 주소를 이용                     |


## 6. Data center networking

구글, 마이크로소프트, 아마존 모두 아주 거대한 data center 를 지었다. 이 data center 내부에서는 각자의 data center network 가 있다. 여기서의 챌린지는, (1) 다수의 어플리케이션이 정말 많은 양의 클라이언트를 서빙하고 있다, (2) 로드를 밸런싱 하는 것 이다. 

### Load balancer 

구글, 마이크로소프트 등 다양한 어플리케이션을 제공한다. 외부에서 클라이언트가 IP 주소를 이용해 요청을 보내면, 첫번째로 이 요청은 데이터 센터 내부의 load balancer 에게로 간다. Load balancer 는 이 요청을 호스트에게 분배하고, 호스트끼리의 로드의 균형을 맞춘다. 외부의 클라이언트는 데이터센터 내부에서 어떤 일이 일어났는지 모른채로 응답을 받는다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b559d463-c976-4194-943f-378da0f4031c">

### Hierachical Architecture

수천개 이상의 호스트로 스케일링 하기 위해서는 데이터 센터는 hierarchy of routers and switch 를 도입한다. 꼭대기에는 border 라우터가 있어서 access 라우터와 연결한다. 각 access 라우터 밑에는 3-tier switch 가 있다. 각 access 라우터는 top-tier 스위치에 연결하고, 각 top-tier 스위치는 다수의 second-tier 스위치들과 load balancer과 연결한다. 각 second-tier 스위치는 rack 의 **TOR(Top of Rack)** 을 통해 다수의 rack 과 연결한다. 모든 링크들은 주로 이더넷을 사용한다. 

예시 데이터 센터 구조는 아래와 같다.

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b017322d-452a-423e-9313-2103a1f04121">

#### Cloud Applications

클라우드 어플리케이션은 웹서핑, 광고, 추천 등등을 제공한다. 데이터 센터 네트워크로부터 3가지를 요구하는데, (1) Low latency, (2) High burst tolerance, (3) High utilization for long flows 이다. 

#### Edge Computing

IoT 와 mobile 에 대응해서 나온 새로운 솔루션이 edge computing 이다. 이것은 네트워크의 edge 에서 hyper-local storage, processing captacity 를 제공하는 작고, 분배된 데이터 센터이다. 데어터가 나오는 곳에 가깝게 컴퓨팅 리소스를 배치한다. 

Edge computing 은, (1) 전송 지연을 줄일 수 있고, (2) 움직이는 기구에 설치할 수 있다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.6
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 