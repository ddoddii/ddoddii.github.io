+++
author = "Soeun"
title = "[네트워크] Link Layer and LANs"
date = "2023-11-18"
description = "Link Layer 에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "link-layer-and-LANs"
+++

## 1. Introduction to Link Layer

link-layer 프로토콜에서 작동하는 디바이스(host, router)를 node 라고 하자. 근접한 node 들이 소통하는 통로를 link 라고 하자. datagram 이 발신자 호스트에서 수신자 호스트로 도달할 떄까지, ene-to-end path 내의 개별 링크 를 따라 가야 한다. 링크에는 wired link, wireless link, ethernet link, LAN 등이 있다. 링크 내에서, 전송하는 노드는 datagram 을 link-layer frame 으로 캡슐화 한 다음에, 링크를 통해 전송한다. 

data link layer 는 하나의 노드에서 물리적으로 근접한 다른 노드들에게 datagram 을 전송하는 책임이 있다. 

datagram 은 다른 링크들 내에서 각각 다른 link protocol 에 따라 전송된다. 그리고 각 프로토콜은 다른 서비스들을 제공한다. 

### 1.1. The services provided by the Link Layer

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

### 1.2. Where is the Link Layer implemented?

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

### 3.1. Channel Partitioning Protocols

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

### 3.2. Random Access Protocols

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

Slotted ALOHA 의 efficiency를 계산해보자. N 개의 노드가 보내야할 많은 frame 들이 있고, 각 노드는 확률 p 로 slot 에 전송한다. 하나의 노드가 slot 에서 success 할 확률은 $p(1-p)^{N-1}$ 이다. 즉, 특정 노드만 보내고 나머지 노드들은 전송하지 않는 경우이다. Any 노드가 성공할 확률은 $Np(1-p)^{N-1}$  이다.

여기서 max efficiency 는 , $Np(1-p)^{N-1}$ 를 최대화하는 $p^*$ 을 구하면 된다.  $Np(1-p)^{N-1}$ 의 리밋을 구하면, max efficiency 는 $1/e=3.7$ 이 된다. 즉, 가장 이상적일 때, 채널은 37% 의 확률로 전송에 성공한다. 


#### Pure (unslotted) ALOHA 

unslotted ALOHA 는 synchonization 기능이 없다. 즉, frame 을 slot 의 시작시간에 맞추서 전송하는 것이 아니라 frame 이 도착하면 바로 전송한다. 하지만 이때 충돌 확률이 증가한다. $t_{0}$ 에 보내진 frame 은 $[t_{0}-1,t_{0}+1]$  시간에 보내진 frame 과 충돌한다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cfc0be22-252c-4de8-b634-4d8583253656">

Pure ALOHA 의 efficiency 를 계산해보자. 
$P(success\ by \ given \ node) =  P(node\ transmits)*P(no\ other\ node\ transmits \ in [t_{0}-1,t_{0}])*P(no\ other\ node\ transmits \ in [t_{0},t_{0}+1]) \n$ $=p*(1-p)^{N-1}*(1-p)^{N-1}$
$=p*(1-p)^{2(N-1)}$

n 을 리미트로 보내고, 최적화된 p 를 고르면, max. effiency 는 0.18 이 나온다. 즉, clock synchro 에 들어가는 overhead 는 없어졌지만, 효율을 오히려 slotted ALOHA 보다 나빠졌다 !

#### CSMA(Carrier Sense Multiple Access)

ALOHA 에서는 패킷을 전송하는 것은 노드의 결정이었다. 즉, 다른 노드들이 전송하고 있건 말건 신경쓰지 않는다. CSMA 는 전송하기 전에 "listen" 하는 단계가 있다. 이것을 **<span style="background:#FEFBD1">carrier sensing</span>** 이라고 한다. 만약 채널이 idle 하다면, 전체 패킷 을 전송한다. 전송하는 와중에도, 다른 노드가 패킷을 전송하는 것을 감지한다면 (**<span style="background:#FEFBD1">collision detection</span>**) 전송하는 것을 멈춘다. 

모든 노드들이 carrier sensing 을 하는데, 어떻게 충돌이 일어나는가? 아래의 다이어그램에서 볼 수 있다. $t_0$ 에 B 는 채널이 idle 하다는 것을 알고, 전송을 시작한다. $t_1$ 에, D 는 전송할 패킷이 생긴다. B 는 이때 여전히 전송중이지만, B 가 전송한 패킷의 bit 가 아직 D까지 도착하지 않았다. 따라서 D는 $t_1$ 에 채널이 idle 하다고 느낀다. 따라서 D 는 패킷을 전송하기로 시작한다. 조금 후에, B 는 D의 전송과 interfere 하기 시작한다. 따라서, end-to-end **<span style="background:#FEFBD1">channel propagation delay</span>**(하나의 노드에서 다른 노드들로 propagate 하는데 걸리는 시간) 가 성능에 굉장히 중요한 역할을 한다는 것을 알 수 있다. 

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

**<span style="background:#FEFBD1">Binary exponential backoff</span>** 알고리즘은 이더넷에 의해 사용되는데, 이 문제를 해결해준다. 이미 n번의 collisions 를 경험한 frame 을 전송할 때는, $\{0,1,2,...,2^n-1\}$ 에서 랜덤한 값 K 를 선택한다. 이렇게 하면 충돌한 횟수가 커질수록 선택하는 구간의 폭도 커진다. 

그렇다면, CSMA/CD 의 efficiency 를 보자. 
$T_{prop}$ = max  propagation  delay  between  2  nodes in LAN
$t_{trans}$ = time to transmit max-size frame

$efficiency= \frac{1}{1+5t_{prop}/t_{trans}}$  

$t_{prop}$ 가 0 으로 수렴하고, $t_{trans}$ 가 무한대로 가면, efficiency 는 1로 수렴한다. 즉, ALOHA 보다 더 좋은 성능을 낸다. 

### 3.3. Taking-Turns Protocols

앞에서, multiple access protocol 의 이상적인 조건들은 다음과 같았다.  (1) 하나의 노드만 전송하면, R 의 속도로 전송한다. (2) M 노드들이 active 하다면 각 노드들은 R/M bps 의 속도로 전송한다. 앞에서 본 CSMA, ALOHA 는 첫번째 조건은 만족하지만 두번째 조건은 만족하지 않는다. 따라서 새로운 프로토콜인 **<span style="background:#FEFBD1">Taking-Turns protocol</span>** 이 나왔다.  구체적으로 구현하는 프로토콜은 여러가지이지만, 여기서는 먼저 polling protocol 에 대해 알아보자. 

#### Polling Protocol

여기서는 하나의 노드가 마스트 노드로 지정된다. 마스터 노드는 slave 노드에게 전송하라고 초대(poll)한다. 예를 들어, 마스터 노드는 1번 노드에게 메세지를 보내서 frame 을 전송하라고 말한다. 그 후 다른 노드들에게도 이 작업을 반복한다. 마스터 노드는 노드가 frame 전송을 끝냈는지 채널의 시그널을 관찰해서 알 수 있다. 

Polling protocol 은 충돌과 빈 slot 이 있는 것을 방지한다. 하지만 단점도 있다. 첫번째는 , polling delay 가 있다는 점이다. 예를 들어 하나의 노드만 active 하다면, 그 노드는 R bps 보다 작은 속도로 전송한다. 왜냐하면 마스터 노드는 각각 inactive 노드들에게 active 노드가 frame 전송이 끝난 후 메세지를 보내야 하기 때문이다. 두번째는, 마스터 노드가 fail 하면 전체 프로토콜이 fail 한다. 

<img width="300" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7db6edeb-6286-4284-a398-9517a1cb7989">

#### Token-passing protocol
여기서는 마스터 노드가 없다. 대신, 노드끼리 **토큰**을 전달한다. 이때 전달은 고정된 순서를 따른다. 만약 노드가 토큰을 건내 받으면, 전송한 frame 이 있을 때만 토큰을 가지고 있는다. 만약 frame 이 없다면 다른 노드에게 토큰을 전달한다. 

하지만 단점이 있다. token overhead 가 발생할 수 있고, 토큰이 사라지거나 하나의 노드가 망가지면 전체가 무너질 수 있다. 

<img width="300" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d6e7da65-dbb5-4fb1-8d14-78df44f2d8ee">


### 3.4. DOCSIS : The Link-Layer Protocol for Cable Internet Access

Cable internet 에서 위에서 알아본 프로토콜이 어떻게 작동하는지 알아보자. Cable network 는 수천개의 가정에서 cable modem 들을 연결하여 CMTS(cable modem termination system) 에 연결한다. DOCSIS(Data-Over-Cable Service Interface Specifications) 는 이것을 결정한다. 

<img width="500" alt="IMG" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/66259853-3351-41bf-922d-2d0fb5928373">

