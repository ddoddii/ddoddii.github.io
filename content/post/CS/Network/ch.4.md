+++
author = "Soeun"
title = "[네트워크] Network layer - Data plane"
date = "2023-10-30"
description = "네트워크 레이어에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "network-layer"
+++

## 1. Overview of Network Layer

Host1 이 Host2 에게 데이터를 전송하고, 사이에 여러개의 라우터가 있다고 하자. H1의 network layer에서는, transport layer 에서 segment 를 받아서 datagram으로 캡슐화 후 근처 라우터(R1)에게 보낸다. H2 의 network layer에서는 이 datagram 을 근처 라우터(R2) 에게서 받아서 transport-layer segment 를 분리 한 후 transport layer 로 전달한다. 라우터의 역할은 datagram 을 input port 에서 output port 로 forwarding 하는 것이다. Network control plane 의 역할은 이 로컬 라우터들에서 datagram 이 처음부터 끝까지 잘 전달되는 것이다. 

### Forwarding and Routing : The Data and Control planes

- **Forwarding**

라우터의 input link 에 패킷이 도착하면, 라우터는 이 패킷을 적절한 output link 에게 보내야 한다. 라우터 내부에서 행해지는 행동이다. 상대적으로 짧은 시간(nanoseconds) 내에 결정된다.

- **Routing**
  
네트워크 레이어에서는 패킷이 sender -> receiver 에게 갈 동안의 루트를 결정해야 한다. 이 루트는 routing algorithm 에 의해 계산된다. end-to-end 시점에서 계산되는 것이다. 상대적으로 forwarding 보다 긴 시간(miliseconds)이 걸린다. 

모든 라우터에는 forwarding table 이 있다. 라우터는 도착하는 패킷의 헤더를 검사해서 이 헤더의 값들을 바탕으로 forwarding table 내에서 계산한다. 

#### Control plane : Two Approaches

<span style="font-size:110%"><span style="background-color: #EBFFDA">**The Traditional Approach**</span></span>  

routing algorithm 이 각 라우터의 내부에 있다. 각 라우터는 다른 라우터와 소통(routing message)해서 다른 라우터의 라우팅 알고리즘 정보를 받고, forwarding table 내에서 계산한다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e126dfd2-a59b-4c49-9736-6d25c152c372">


<span style="font-size:110%"><span style="background-color: #EBFFDA">**The SDN Approach**</span></span>

SDN(Software Defined Networking) 에서는 remote controller 가 라우터에 의해 이용되는 forwarding table 을 계산한다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/381d4d6f-621f-44fa-bafd-6be71ee88037">

### Network Service Model

네트워크 레이어에서 제공할 수 있을법한 서비스들을 보자. 가능한 서비스들은 아래 목록이다.
- Guaranteed delivery
- Guaranteed delivery with bounded delay
- In-order packet delivery
- Guaranteed minimal bandwidth
- Security

하지만, 네트워크 레이어는 단 하나의 서비스 - **best effort service** 만 제공한다. 위의 것 중 어느 것도 보장하지 않는다. 그저 최대한의 노력을 한다. 그러면 best effort 가 no services at all 을 의미할 수 있지 않냐 ? 생각할 수 있는데, 인터넷의 best effort service 는 놀랍게도 충분하다. 

## 2. What's inside a router?

라우터의 4가지 구성요소를 보자. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/babfcef3-ad49-4d23-acbe-986cdd7a8688">

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Input ports**</span></span>  

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f08835d6-da6e-4190-ab70-edb09704d080">

- Line termination
- Link layer protocol
- lookup, forwarding , queueing
  - Destination-based forwarding
    IP 주소만 가지고 forwarding 을 한다. 즉 목적지만 가지고 결정한다. 
  - generalized forwarding
    헤더 필드 값을 기반으로 forwarding 한다. 목적지 외에 도 많은 값들을 고려해서 결정한다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Switching fabric**</span></span>  

라우터의 input port 와 output port 를 연결한다. 라우터 안의 네트워크 이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Output ports**</span></span>  

switching fabric 으로부터 받은 패킷을 저장하고, 이 패킷을 outgoing link 를 통해 내보낸다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Routing processor**</span></span>  

라우팅 프로세서는 control-plane 기능을 한다. 전통적인 라우터에서는, 라우팅 프로토콜을 실행하고, 라우팅 테이블을 저장하고 있다. 
SDN 라우터에서는 라우팅 프로세서는 remote controller 와 소통해서 계산된 fowarding table 정보를 받아온다. 

### Input port processing and Destination-Based Forwarding

Input port 의 line termination 과 link-layer processing 은 input link 의 물리적인 부분을 담당한다. lookup 은 라우터의 기능에서 핵심적인 부분을 담당한다. 라우터가 forwarding table 을 이용해서 어느 output link 로 내보낼지 결정하는 곳이다. forwarding table 은 routing processor 또는 remote SDN controller 에 의해 계산되고 업데이트 된다.

예시를 통해 알아보자. 라우터가 4개의 link (0,1,2,3) 가 있다고 하자. 아래는 forwarding table 이다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7740cee3-2d5b-4d99-b029-a0d5ca4e8c8b">

위의 범위로 나타낸 것을 단순화 시켜서 나타내면, 아래 처럼 **prefix** 로 나타낼 수 있다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1e9b6f3b-639c-4d05-920f-3ec067013f81">

라우터는 패킷의 destination address 를 테이블의 prefix 와 매칭한다. 예를 들어, packet의 목적지 주소가 11001000 00010111 00010110 1010001 이라고 하자. 이 주소는 위의 table 에서 봤을 때 Link interface 0 과 매치한다. 하지만, 다수의 prefix 와 매칭이 되면 어떻게 할까? 이 때는 **longest prefix matching rule** 을 사용한다. 가장 길게 일치하는 prefix 가 있는 링크로 보내진다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6f31c7c3-3479-4e49-8f41-1cdc90adee6b">

### Switching

lookup 을 통해 패킷의 output port 가 정해졌으니, 패킷은 switching fabric 으로 보내진다. Switching 은 여러 가지 방법이 있다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/52cb241a-dc59-457a-a975-43ebb5ede893">

- **Switching via memory**
  
  Switching 이 CPU (router processor) 의 관리 하에 이루어진다. input port 에 패킷이 도착하면 라우터 프로세서에 알림이 간다. 패킷은 그러면 input port 에서 프로세서 메모리로 복사된다. 라우팅 프로세서는 헤더에서 목적지 주소를 뽑아내고, forwarding table 에서 적합한 output port 를 계산한다. 그 후 패킷을 output port 의 버퍼에 복사한다. 이 경우에 memory bandwidth 이 B packets / sec 이면, fowarding throughput 은 B/2 보다 작아야 한다. 
  

- **Switching via a bus**
  
  라우팅 프로세서의 간섭 없이, input port 는 패킷을 shared bus 를 이용해서 output port 로 보낸다. 여기서 input port 는 이 패킷이 어느 output port 로 갈지를 말해주는 switch-internal label(헤더) 를 패킷의 앞에 부착하고, 패킷을 버스에 전달한다. 모든 output port 는 패킷을 전달받만, 라벨과 매치하는 포트만 패킷을 저장하고, 나머지 포트들은 패킷을 버린다. 이 라벨은 output port 에서 제거된다. 만약 여러 개의 패킷이 라우터에 동시에 도착하면, 이 패킷이 서로 다른 input port 에 도착한다고 해도 하나 빼고 다른 모든 패킷들은 기다려야 한다. 왜냐면 하나의 패킷만 버스를 건널 수 있다. 따라서 라우터의 스위칭 속도는 버스의 속도에 제한된다. 

- **Switching via an interconnection network**
  
  위에 제한을 극복하기 위해, 더욱 복잡한 네트워크가 소개되었다. crossbar switch 는 2N 개의 버스들로 구성되고, N input ports 를 N output ports 에 연결한다. 수직, 수평 버스는 crosspoint 에서 만나는데, switch fabric controller 에 의해 open / close 될 수 있다. 
  
  아래 예시에서, A -> Y 로 보내고 동시에 B -> X 로 보내고 싶다고 하자. 두 경로가 겹치지 않으므로 동시에 보낼 수 있다. 이것을 <span style="background-color: #FEFBD1">**non-blocking**</span> 이라고 한다. 그러나 여전히 다른 input port 에서 온 2개의 패킷이 같은 output port 로 가고 싶을 때는, 하나는 input 에서 기다려야 한다. 왜냐면 버스는 딱 한개의 패킷만 전송할 수 있기 때문이다. 

<img width="300" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ec02b966-f599-4db8-9087-390d97b44bc7">

### Where Does Queuing Occur?

Queuing 은 input port, output port 둘 다에서 발생할 수 있다. 만약 queue 가 커지면, 라우터의 메모리가 꽉 차게 되어 packet loss 가 발생할 수 있다. 패킷이 네트워크 내부에서 잃어버리게 되거나, 라우터에 드랍되는 경우는 이러한 경우이다. 

#### Input Queuing
만약 fabric 이 처리하는 속도가 input port 에 패킷이 들어오는 속도보다 느리면, input queue 가 발생할 수 있다. input buffer 가 꽉 차서 queuing delay 와 packet  loss 가 발생할 수 있다. 

N개의 input, N개의 output ports 가 있고, input , output line speed 가 $R_{line}$ 이라고 하자. Switching fabric transfer rate 를 $R_{switch}$ 라고 하자. 이 때 $R_{switch} > N*R_{line}$ 이면 무시할 수 있는 queuing 만 발생한다. 그러나, $R_{switch}$ 가 충분히 빠르지 않으면 어떻게 될까? 이 때 input port 에서 packet queuing 이 발생할 것이다. 이 queuing 의 결과를 설명하기 위해서, 상황을 가정하자. crossbar switching fabric 에서, (1) link speed 는 모두 동일하고, (2) 하나의 패킷이 input port 에서 output port 로 전달되는 속도는 input port 에서 패킷을 받는 속도와 같고, (3) 패킷은 input queue 에서 output queue 로 FCFS(First-come-First-served) 방식으로 보내진다. 다수의 패킷은 output port 이 다를 경우에 병렬적으로 동시에 보내질 수 있다. 하지만, output port 가 같은 경우, 하나가 보내질 동안 나머지는 input queue 에서 기다려야 한다. 

아래의 경우를 보자. 왼쪽 상황에서 빨간 패킷 2개가 같은 output port 로 가고자 하므로 하나의 패킷은 기다려야 한다. 이때, 맨 아래 빨간 패킷이 기다려야 하면, 그 뒤에 있는 초록 패킷도 기다려야 한다. 이 초록 패킷은 초록 output port 에 가는 경쟁 패킷이 없음에도 기다려야 한다. 이것을 <span style="background-color: #FEFBD1">**head-of-line (HOL) blocking**</span> 이라고 한다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8ee26417-af87-4e7c-9375-e71bcc17baa0">

#### Output Queuing

Output port 는 아래와 같이 이루어져 있다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/edf79784-af0e-4c95-a664-37d2d8427e90">

만약 transmission rate 보다 fabric 에서 패킷이 도착하는 속도가 더 빠르다면, <span style="background-color: #FEFBD1">**buffering**</span> 이 필요하다. 이때 버퍼가 가득 차면, 도착하는 패킷을 버릴지, 이미 queue 에 있는 패킷을 버릴 지 선택해야 한다. 어느 때는 버퍼가 다 차기 전에 미리 버퍼 안의 패킷을 버리는 것이 더 이득일 수 도 있다. 이것을 <span style="background-color: #FEFBD1">**active queue management(AQM)**</span> 이라고 한다. 

이때, 얼마나의 버퍼링이 필요한지 계산할 수 있다. link capacity 가 C이고, N개의 flow 가 있을 때, 버퍼링은 $RTT*C/\sqrt(N)$  이다.   

### Packet Scheduling

<span style="background-color: #FEFBD1">**Scheduling discipline**</span> 은 queue 에 있는 패킷 중 어느 것을 먼저 전송할 지 선택하는 메카니즘이다. 크게 3가지 종류가 있다. 

- **FIFO (First in First out) scheduling**
  
  도착한 순서대로 낸보낸다. 이때, queue 가 꽉 차면 어떤 패킷을 버릴 것인가? tail drop, priority, random drom 중에 선택할 수 있다. 
  
  <img width="355" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/eb489b59-9dc1-40d0-9345-70f674313071">
  
- **Priority Queuing**
  
   패킷이 도착하면, 우선순위에 따라 클래스가 나누어진다. 가장 높은 우선순위가 있는 패킷을 전송한다. 아래 예시에서, **non-preemptive priority queuing** 원칙에 따라 순위가 낮은 2번 패킷이 전송 중에 순위가 높은 4번 패킷이 도착해도 전송 중인 패킷은 방해하지 않는다. 
  
  <img width="333" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/33fa3e17-6e37-41ba-92ce-a93136e87858">

- **Round Robin(RR) scheduling**
  
  패킷이 도착하면 여러 개의 클래스에 저장된다. RR 는 우선순위 대신, 여러 클래스를 돌아가며 전송한다. A클래스 패킷 -> B클래스 패킷 -> C클래스 패킷 -> A클래스 패킷 ->... 이런 식으로 순환한다. 
  
  <img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f64c2fa5-4c97-4261-9c35-47b86f004f75">

- **Weighted Fair Queuing(WFQ)**

  일반화된 형태의 Round Robin 이다. 여기서도 패킷들은 클래스로 분류된다. 각 클래스는 사이클마다 weighted amount 의 서비스를 받는다. 어느 인터벌에도 class i 는 bandwidth 의 $w_{i} / \sum w_i$  만큼 할당을 받는다. 
  
  <img width="432" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/ee296ad0-2d6a-4ec4-91ed-bd037c23429e">

## 3. IP : Internet Protocol

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/59c90956-bcfb-4cc4-bc9e-67b53360bd3e">

현재는 2가지 버전의 IP 가 있다. - IPV4, IPV6

### IP Datagram format

<span style="font-size:110%"><span style="background-color: #EBFFDA">**IPV4**</span></span>  

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/63065718-b7bb-4eb1-a83e-0380dcdd1bb8">

IP datagram 은 옵션이 없을 때 20byte 의 헤더를 가진다. 만약 datagram 이 TCP segment 를 가진다면 , datagram 은 총 40byte 의 헤더를 가진다. 

### IPv4 Datagram Fragmentation

네트워크 링크 프레임이 전송할 수 있는 가장 큰 용량의 데이터는 <span style="background-color: #FEFBD1">**MTU(Maximum transfer size)**</span> 로 정해진다. 라우터에서 다른 라우터로 전송되기 위해 IP datagram 이 캡슐화 되면서 헤더가 붙기 때문에 IP datagram 의 길이에 강한 제한을 둔다. 문제는 보내는이와 받는이 사이에 루트에 있는 링크에서 다른 link-layer protocol 을 사용해서 MTU 가 다를 때이다. 

만약 하나의 링크에서 N 사이즈의 datagram 을 받았는데, 보내야 하는 링크의 MTU 가 N 보다 작으면 어떻게 할 것인가? 이때를 대비해서 IP datagram 에 있는 payload 를 더 작은 여러 개의 IP datagram 으로 나누는 것이다. 그리고 이 작은 IP datagram 을 각각 캐슐화 해서 이것들을 outgoing link 로 보내는 것이다. 이 작은 datagram 들을 <span style="background-color: #FEFBD1">**fragment**</span> 이라고 부른다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/def3b76b-23d9-45f7-bb14-22981bac6105">

각 fragment들은 목적이의 transport layer 에 도달하기 전에 재조립(reassemble)되어야 한다. 
TCP, UDP 는 완전한 segment 를 기대하지 분해된 segment 가 도착하길 기대하지 않는다. 하지만 네트워크 코어를 간단하게 유지하자는 원칙 아래, IPv4 설계자들은 재조립 과정을 end system 에 맡기기로 했다. 

목적지의 호스트가 같은 source 에서 온 여러 개의 datagrams 를 받았으면, 이 datagrams 들이 모두 같은 원본의 datagram 에서 왔는지 판별해야 한다. 또, 마지막 조각을 받았을 때 이 조각들을 원본으로 어떻게 재조립할지도 파악해야 한다. 

목적지의 호스트가 이 작업을 하기 위해서, IPv4 의 설계자들은 IP datagram header 에 **identification, flag, fragmentation offset** 필드를 추가했다. 보내는 호스트는 각 datagram 의 조각을 전송할 때마다 id 를 1씩 증가시킨다. 목적지의 호스트가 모든 datagram 들을 다 받았음을 확신하기 위해, 마지막 datagram 의 flag 는 0 으로 표기한다. 나머지 datagram 들의 flag 는 모두 1이다. 

### IPv4 Addressing

IP addressing 을 논의하기 전에, 호스트와 라우터들이 인터넷에 어떻게 연결되어 있는지를 보자. 호스트는 네트워크 안에 하나의 링크가 있다. 호스트 안의 IP 가 datagram 을 전송하고 싶을 때, 이 링크를 이용한다. 호스트와 물리적인 링크 사이의 경계를 <span style="background-color: #FEFBD1">**interface**</span> 라고 한다. 라우터의 역할이 하나의 링크로부터 datagram 을 받고 이것을 다른 링크로 전송하는 것이기 때문에, 라우터는 보통 여러 개의 interface 가 있다. 호스트는 보통 1개 또는 2개의 interface 가 있다.  호스트와 라우터가 IP datagram 을 주고 받을 수 있기 때문에, IP 는 모든 호스트와 라우터의 인터페이스가 고유의 IP 주소가 있기를 요구한다. 따라서, <span style="background-color: #FEFBD1">**IP 주소는 인터페이스와 관련**</span>이 있지, 그 인터페이스를 포함하는 호스트 또는 라우터와 관련이 있는 것은 아니다. 

각 IP 주소의 길이는 32-bits 이다. IP 주소는 dotted-decimal notation 인데, 8 bit 로 나누어진 4개 파트로 되어 있다. 예를 들어, IP 주소가 193.32.216.9 이면 이진수로 표현하면 1100001 001000000 11011000 00001001 이다. 

<img width="357" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/42d8c67f-3265-4d76-8ae9-9b27b452aeda">

위의 예시에서, 3개의 인터페이스를 가진 하나의 라우터에 7개의 호스트가 연결되어 있다. 위의 왼쪽에 있는 3개의 호스트는 모두 223.1.1.xxx 의 IP 주소를 가지고 있다. 이것은 처음 24bit 의 IP 주소가 일치한다는 것이다.  이 4개 (왼쪽 호스트3개 + 라우터의 인터페이스1개) 의 인터페이스는 이더넷에 의해 서로에게 연결되어 있다. 이렇게 연결하는 네트워크를 <span style="background:#FEFBD1">**subnet**</span> 이라고 한다. 

IP addressing 은 이 subnet 에게 223.1.1.0/24 의 IP 주소를 부여한다. /24 는 <span style="background:#FEFBD1">**subnet mask**</span> 로, 32-bit 의 IP 주소 중에 왼쪽 24-bit 를 뜻한다. 즉, 223.1.1.0/24 Subnet 안에 있는 IP 주소는 223.1.1.xxx 의 IP 주소를 가져야 한다. 

subnet 을 정의하기 위해서는, 각 인터페이스를 호스트 또는 라우터와 분리해서 고립된 섬과 같은 네트워크를 만든다. 이 고립된 섬을 subnet 이라고 정의한다. 

인터넷의 주소 할당 전략은 <span style="background:#FEFBD1">**CIDR(Classless Interdomain Routing)**</span>이라고 한다.  주소 포맷은 a.b.c.d/x 인데, x 는 subnet 부분에 있는 bits 의 개수이다. 아래와 같이 Subnet 부분의 bit이 23개이면, 200.23.16.0/23 으로 나타낸다. 앞에 23bit 를 이용해서 organization 을 특정하고, 뒤의 9 bit 를 이용해서 organizaiton 안에 어떤 호스트인지를 특정한다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8e2bbaf8-4a8b-4c4b-a0cc-7c83510f6f1a">


이제 IP addressing 을 알아봤으니, 애초에 처음부터 subnet 과 호스트가 어떻게 IP 주소를 받는지 알아보자.

#### IP addresses : how to get one?

조직의 subnet 을 위해 IP 주소를 얻고 싶을때는, ISP 를 연락한다. 그러면 ISP 가 IP 주소를 할당해준다. 그렇다면 ISP 그 자체는 IP 주소를 어떻게 얻을까? ICANN(Internet Corporation for Assigned Names and Numbers) 가 IP 주소를 전세계적으로 관리한다.

조직이 IP 주소의 블럭을 얻고 나서는, 호스트가 개별 IP 주소를 얻을 수 있다. 이때, **<span style="background:#FEFBD1">Dynamic Host Configuration Protocol(DHCP)</span>** 를 사용한다. DHCP 는 호스트가 IP 주소를 자동으로 얻을 수 있게 한다. 네트워크 관리자는 호스트가 매번 네트워크에 접속할 때마다 같은 IP 주소를 할당할 수 있고, 아니면 임시 IP 주소를 할당할 수 도 있다. DHCP 가 IP 주소를 자동으로 할당하기 때문에, 이것을 <span style="background:#FEFBD1">**plug-and-play**</span> 또는 <span style="background:#FEFBD1">**zeroconf**</span> 프로토콜이라고 한다. 

네트워크가 IP 주소의 subnet 파트를 어떻게 얻을 수 있을까? 바로 ISP 주소의 부분을 할당받는다. 이때 **<span style="background:#FEFBD1">Hierarchical addressing</span>** 을 이용한다.

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8892c584-24eb-4e75-810e-0624f0a30aad">



#### Dynamic Host Configuration Protocol

DHCP 의 목적은, 호스트가 네트워크에 접속했을 때 자동으로 IP 주소를 할당하는 것이다. DHCP 는 IP 주소를 재사용할 수 있게 하는데, 접속이 되어있을 때만 주소가 유효하다. 

DHCP 는 **서버-클라이언트 프로토콜**이다. 클라이언트는 네트워크에 접속하고자 하는 호스트이고, 서버는 네트워크 서버이다. DHCP client-server 시나리오를 살펴보자. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fd37ebbc-6b7b-42cb-addc-c18953e4755b">

DHCP 클라이언트-서버 시나리오는 크게 4가지 단계로 이루어진다. 

1. Host broadcasts **"DHCP discover"** msg
   
   네트워크에 새롭게 접속한 호스트는 DHCP discover msg 를 UDP 패킷의 포트 67 로 브로드캐스팅한다. 
   
2. DHCP server responds with **"DCHP offer"** msg
   
   DHCP discover msg 를 받은 서버는 DHCP offer msg 를 브로드캐스트한다. 서버의 offer message 는 transaction ID, proposed IP, network mask, IP address lease time 으로 구성된다. 
   
3. Host requests IP address : **"DHCP request"** msg
   
   호스트는 서버의 offer msg 중에 하나를 선택해서, 파라미터를 다시 확인하는 ack 를 포함하는 DHCP request msg 를 보낸다. 
   
4. DHCP server sends address : **"DHCP ack"** msg
   
   서버는 DHCP request msg 를 DHCP ack msg 로 응답한다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5ac9cafa-1cb3-45ad-9a10-8f419e9df3a4">

클라이언트가 DHCP ACK 를 받으면, 상호작용은 끝나고 클라이언트는 DCHP 를 통해 할당받은 IP 주소를 lease duration 동안 쓸 수 있다.

DHCP 는 IP 주소만 할당하는 것이 아니다. 클라이언트에게 first-hop 라우터의 주소, DNS 서버의 이름과 IP 주소, network mask 를 줄 수 있다. 예시를 보자. 

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b2dab63e-acf0-4b66-b01e-236282fc133e">

연결하고자 하는 랩탑은 IP 주소, first-hop 라우터의 주소, DNS 서버의 주소를 원한다. DHCP request 는 UDP -> IP -> 이더넷에 차례대로 캡슐화된다. 이더넷은 이것을 브로드캐스팅 하고, DHCP 서버를 운영중인 라우터에 도착한다. 이더넷 -> IP -> UDP -> DHCP 로 demux 된다.

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1f82406a-6e12-4491-a196-6b1ad18708bd">

DHCP 서버는 클라이언트의 IP 주소, first-hop 라우터의 주소, DNS 서버의 이름 & 주소가 담긴 DHCP ACK 를 만든다. 이것이 캡슐화되어서 다시 클라이언트에게 도착한다.

### Network Address Translation(NAT)

오피스에서 규모가 커져서 더 많은 IP 주소가 필요할 때가 생길 수 있다. 그러나 이미 ISP 에서 할당된 블럭의 뒷 부분의 IP 주소를 다른 곳에 할당했으면 어떻게 할까? 다행히도, 이런 시나리오에서 IP 주소를 할당하는 방법이 **<span style="background:#FEFBD1">Network Address Translation(NAT)</span>** 이다. 


<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8113ffa0-c73e-48b0-818a-a16d8d36f9e4">

집이나 오피스에 있는 NAT-enabled 라우터의 예시를 보자. 위의 예시에서, 하나의 라우터에 4개의 인터페이스가 연결되어 있는데, 모두 10.0.0/24 의 같은 Subnet 주소를 가진다. 이 subnet 주소 안에서도, 10.0.0.0/8 은 <span style="background-color: #FEFBD1">**private network**</span> 또는 **realm with private addresses** 로, 저장되어 있다. Private network 는 그 네트워크 안에서만 주소가 유효하다는 뜻이다. 전세계에는 수백만 개의 집 네트워크가 있는데, 모두 같은 주소 space (10.0.0.0/24) 를 사용한다. 집 네트워크 내부의 디바이스는 서로에게는 이 네트워크 안에서 패킷을 전송할 수 있다. 그렇지만 집 네트워크를 넘어서, 글로벌 네트워크에서는 어느 집 네트워크 주소인지 구분하지 못할 것이다. 그렇다면, 어떻게 하나의 집 네트워크를 특정할 수 있을까? 이때 사용되는 것이 <span style="background-color: #FEFBD1">**NAT**</span> 이다.

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6cf8ca95-c333-4986-8f07-d4937f217e7d">

NAT-enabled 라우터는 바깥 세상에 라우터 처럼 보이지 않는다. NAT 라우터는 바깥 세상에 <span style="background-color: #FEFBD1">**하나의 IP 주소를 가진 하나의 디바이스 처럼 작동**</span>한다. 집 라우터를 떠나는 모든 트래픽은 모두 고정된 IP 주소(138.76.29.7)를 갖는다. WAN 으로부터 NAT 라우터에 도착하는 모든 Datagram 도 같은 목적지 IP 주소(138.76.29.7)를 갖는다.  그렇다면, 라우터는 이 Datagram 이 네트워크 내에 어떤 호스트에게 전달되는 것인지 어떻게 알까? 여기서 해결책은 <span style="background-color: #FEFBD1">**NAT Translation table**</span> 을 사용하는 것이다. 

호스트 10.0.0.1 이 웹 서버(port 80, IP 128.119.40.186)에게 웹페이지를 요청한다고 하자. 호스트는 자신에게 임의의 포트 번호(3345) 를 지정하고, 이 datagram 을 전송한다. NAT 라우터는 datagram 을 받고, 새로운 포트 번호 5001 을 할당하고, 새로운 IP 주소(138.76.29.7) 을 할당한다. 그리고 이것을 NAT translation table 에 추가한다. 웹 서버는 이 datagram 을 받고 적절한 응답은 NAT 라우터에게 다시 보낸다. 그렇다면 NAT 라우터는 아까 저장해두었던 NAT translation table 에서 이 쌍을 찾아보고 요청을 보낸 호스트에게 응답을 전달한다.

하지만 NAT 는 의견이 갈린다. 
1. 포트 번호는 프로세스를 구분하기 위한 것이지, 호스트를 구분하기 위한 것은 아니라는 의견이다. 이것에 대한 기술적인 해결책으로 NAT traversal tools 이 나왔다. 
2. 라우터는 layer 3 디바이스이므로 네트워크 레이어까지만 패킷을 프로세싱해야 한다는 것이다. 
3. NAT 는 호스트가 직접적으로 소통해서는 안된다는 규칙을 어긴다. 
4. 클라이언트가 NAT 너머의 서버와 직접 연결하고 싶을 때 어떻게 할 것인지 문제가 생긴다. 


### IPv6

IPv6 가 나오게 된 첫번째 계기는 IPv4 의 32-bit 주소가 거의 다 할당되어서 남은 게 점점 부족해지는 상황이었다. 추가적인 계기로는, IPv4 의 헤더 포맷이 너무 길어서 헤더를 읽는 시간이 오래 걸린다는 것이었다. 

#### IPv6 Datagram format
IPv6 에서 달라진 점을 보자.
- **Expanded addressing capabilities**

  기존 32 bits 에서 128 bits 로 IP 주소의 사이즈를 늘렸다. 
- **fixed-length 40 byte header**

  헤더 길이를 고정해서, IPv4 에 있던 header length field 가 사라졌다.
- **no fragmentation allowed**

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/77438d91-aa3c-49ac-adcf-b295b13eec8e">

- **priority**  : flow 내에 datagram 의 우선순위를 명시
- **flow label** : 같은 flow 내에 있는 datagram 을 명시
- **next header** : 상위 레이어의 프로토콜을 명시 

IPv4 에 있었지만 IPv6 에 없어진 것들을 보자. 
- **Fragmentation / Reassembly**
  
  IPv6 은 분할하고 재조립하는 것을 허용하지 않는다. 분할, 재조립 과정은 중간 라우터에서 하지 못하고 source, destination 에서만 할 수 있다. 만약 outgoing link 에 비해 datagram 이 너무 크면 단순히 "Packet too big" 이라는 ICMP error msg 를 발신자에게 보낸다. 
- **Header checksum**
  
  TCP, UDP 레이어에 checksum 이 있으므로, IP 에서는 이 필드를 없앴다.
- **Options**
  
  옵션 필드는 헤더 바깥에 허용된다. Next hdr 필드가 포인터 처럼 옵션을 가르킨다. 이 옵션 필드를 없애서 헤더를 40-byte 로 고정할 수 있었다.

#### Transitioning from IPv4 to IPv6

IPv4 에서 IPv6 으로 변환해도, 라우터를 한번에 다 끄고 교체할 수 는 없다. 그렇다면 IPv4, IPv6 라우터가 혼합된 상태에서 네트워크는 어떻게 작동할 것인가? 

해결책은 **<span style="background:#FEFBD1">Tunneling</span>** 이다.  IPv6 datagram 은 IPv4 라우터에서 IPv4 datagram 내에 payload 로 인식된다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/530a1540-23f7-4c82-98ed-61be0ac5b2cd">

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a03e4e67-ed2e-4f15-b9c3-6cb4821ee8ed">

## 4. Generalized Forward and SDN

앞에서, 전통적인 라우터의 forwarding 은 패킷의 목적지 주소에 의해 결정된다고 했다. 하지만 전의 섹션에서 layer-3 의 기능을 수행하는 여러 미들박스를 봤다(NAT 라우터). 

목적지 기반 forwarding 은 2가지 스텝에 의해 이루어진다. 매칭 IP 주소를 찾고("match"), 패킷을 특정한 ouput port 로 보내는 것("action") 이다. 조금 더 일반화된 "match-plus-action" 을 보자. "match" 는 다양한 헤더 필드에 의해 이루어질 수 있다. "action" 은 output port 에 보내는 것 외에도, 인터페이스로 balancing packets 를 로드하는 것(load balancing), 헤더 값을 재작성하는 것(NAT) 이 있다. 

각 라우터는 logically centralized routing controller 에 의해 계산되고 분배되는 <span style="background-color: #FEFBD1">**flow table**</span> 을 포함한다. Flow table 는 match-plus-action forwarding table 이다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bbce39f8-1057-43f6-836a-f6d28a4ca84d">

<span style="font-size:110%"><span style="background-color: #EBFFDA">**OpenFLow**</span></span>

일반화된 Forwarding 는 OpenFlow 에 기반을 두고 설명할 것이다. OpenFlow 의 flow table 은 다음의 것들을 포함한다.
- **Pattern** : 패킷 헤더 필드에 있는 match 값
- **Actions** : 매치가 된 패킷에 대해: drop, forward, modify 하거나, 컨트롤러에 보내는 것
- **Priority** : 겹치는 패턴을 명확히 하는 것
- **Counters** :flow table entries 에 매치가 된 패킷의 수 

### Match

아래는 OpenFlow 1.0 match rule 에 매치할 수 있는 11가지 패킷 헤더 필드이다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/509a1459-68b8-45c9-a275-7fc8c8a5bb2b">

flow table 에는 wildcard 가 사용될 수 있다. 예를 들어, 128.119.* . * 는 첫 16bits 가 128.119 이면 매치가 된 것이다.

### Action

여러가지 action 이 수행될 수 있다. 

- **Forwarding** : 패킷이 특정한 output port 로 forwarding 될 수 있다. 
- **Dropping** : 매치가 없는 패킷은 드랍된다.
- **Modify-field** : 10개의 패킷 헤더 필드는 패킷이 forwarding 되기 전에 수정될 수 있다. 

### OpenFlow abstraction

<span style="background-color: #FEFBD1">**Match + action**</span> 은 다른 디바이스들의 행동을 통일한다. 
- Router
	- match : longest destination to IP prefix
	- action : forward out a link
- Switch
	- match : destination MAC address
	- action : forward or flood
- Firewall
	- match : IP address and TCP/UDP port numbers
	- action : permit or deny
- NAT
	- match : IP address and port
	- action : rewrite address and port

예시는 아래와 같다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0737c12b-8a81-4ae7-9446-e69d7a2b3528">


## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.4
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 