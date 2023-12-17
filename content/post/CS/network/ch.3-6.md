+++
author = "Soeun"
title = "[네트워크] Principles of Congestion Control"
date = "2023-10-13"
summary = "Congestion control 의 원인과 congestion의 cost들"
categories = [
    "CS"
]
tags = [
    "network"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "congestion-control"
series = ["Network"]
series_order = 11
+++
{{< katex >}}

## Principles of congestion control

congestion 이란 "**네트워크**가 감당하기 어려울 정도로 다수의 호스트들이 많은 데이터를 전송하는 것" 이다. flow control 과 다르다 !! 

### The cause and Costs of Congestion

#### **시나리오1 : 2 senders, 2 receivers, a Router with Infinite Buffers**

![스크린샷 2023-10-21 오후 10 56 24](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9f6bcb81-8900-4cf3-9806-a6656d9a19f1)

HostA 가 $\lambda_{in}$ 속도로 데이터를 보낸다고 하자. 라우터는 무한한 버퍼 공간을 가지고 있다. 재전송은 없다고 하자. output link capacity 는 R 이다. 

아래 첫번째 그래프는 $\lambda_{in}$ 에 따라 받는 쪽에서의 throughput $\lambda_{out}$ 을 나타낸다. 이 때 , 0~R/2 사이의 sending rate 일 때는 receiver 의 throughput 은 sender 의 sending rate 와 일치한다. 하지만, R/2 보다 커져도 계속 R/2 인데, 왜냐하면 2개의 connection 이 링크를 공유하고 있기 때문이다. 

두번째 그래프는, link capacity 근처로 데이터를 보냈을 때 급격히 증가하는 지연을 볼 수 있다.  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c91523b7-0727-44ff-8013-84aed9afd159)

R 근처의 throughput 을 운용하는 것은 throughput 관점에서는 이상적이지만, delay 관점에서는 최악이다. 여기서 congested network 의 cost 를 찾을 수 있다 -  <span style="background-color: #FEFBD1">**large queuing delays are experienced as the packet-arrival rate nears the link capacity**.</span>


#### **시나리오2 : 2 senders, a router with Finite Buffers**

이번에는 라우터에 유한한 버퍼가 있다. 그리고, Sender 가 타이머가 끝난 패킷을 재전송한다고 하자. 어플리케이션에서 소켓을 통해 데이터를 보내는 속도는 $\lambda_{in}bytes/sec$ 이다. 
하지만, transport layer 가 네트워크 계층으로 segment (original + retransmitted 모두 고려) 를 보내는 속도는 $\lambda\prime_{in}bytes/sec$ 이라고 하자. $\lambda\prime_{in}$ 은 offered load 라고도 불린다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3c7f3034-c8e3-4d60-8373-57b9d22db96e)

<span style="font-size:110%"><span style="background-color: #EBFFDA">**2-1. perfect knowledge**</span></span>

시나리오2 에서는 재전송이 어떻게 이루어지느냐에 따라 퍼포먼스가 달라진다. 첫번째로는, HostA 가 라우터의 버퍼가 비어있는지 완벽하게 알아서 버퍼가 비어있을 때만 패킷을 보낸다고 가정하자.  그러면 loss 가 전혀 발생하지 않을 것이다. 그러면, $\lambda_{in}=\lambda'_{in}$ 이 된다. 이때 그래프는 시나리오1과 같아진다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**2-2. known loss**</span></span>

두번째로, 좀 더 현실적으로 sender 는 패킷이 잃어버린게 확실할 때만 패킷을 재전송한다고 하자. 이 경우에 그래프는 아래와 같다. $\lambda'_{in} = R/2$ 일때, receiver 가 데이터를 받는 속도는 R/3 이 된다. 0.5R units 가 전송되면, 0.333은 original data 이고, 0.166은 retransmitted data 인 것이다.   

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c91b9150-1af2-483a-98a4-dade414af32a">

여기서 congested network 의 두번째 cost 를 알 수 있다 -  <span style="background-color: #FEFBD1">**the sender must perform retransmissions in order to compensate for lost packets due to buffer overflow**.</span>

<span style="font-size:110%"><span style="background-color: #EBFFDA">**2-3. duplicates**</span></span>

이번에는 timeout 이 너무 일찍 되어서 인한 중복 재전송이 있다고 하자. sender 는 타임아웃이 너무 빨리 되서 불필요하게 카피를 보낸 것이다. 이때 receiver 는 중복된 패킷은 버릴 것이다. 이때, 재전송된 패킷을 보내는데 사용된 라우터의 일은 낭비된 것이다. 다른 패킷을 전송할 수 있던 것을 불필요한 패킷을 전송하느라 낭비한 것이다. 아래 그래프에서, 패킷은 2번 전송되었으므로, throughput 는 대략적으로 R/4 가 될 것이다.

<img width="250" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/edd16186-cf30-4046-8b4b-292da71520a9">

여기서 congested network 의 세번째 cost 를 알 수 있다 -  <span style="background-color: #FEFBD1">**unneeded retransmissions by the sender in the face of large delays may cause a router to use its link bandwidth to forward unneeded copies of a packet.**</span>

#### **시나리오3 : 4 Senders, Routers with Finite Routers, Multihop Paths**

이번에는 4개의 host 가 있고, 서로의 데이터를 보내는 경로가 겹친다고 하자. 만약, 빨간색 $\lambda'_{in}$ 이 증가하면, 파란 패킷들은 드랍된다. 따라서 파란색의 throughput 은 0 으로 수렴한다. 

![스크린샷 2023-10-21 오후 11 29 02](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9697dd62-cd06-4fc9-bdcc-b26ae8d62dfc)

이때 그래프를 보면 아래와 같다. 모두 너무 많이 보내면, 결과적으로 드랍되는 패킷들이 생기면서 $\lambda_{out}$ 이 0으로 수렴한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f2fd177b-faed-49c7-8927-e9278eb6e0d3)

D 에서 B 로 패킷을 보냈는데, A,D 보내는 경로가 겹치는 라우터에서 D가 보낸 패킷이 드랍되었다고 하자. 그러면 첫번째 라우터에는 두번째 라우터에서 이 패킷이 드랍될지 모르고 일을 한 것이다. 이것은 낭비된 작업이다. 따라서 여기서 또 다른 cost of congestion 을 알 수 있다 -  <span style="background-color: #FEFBD1">**when packet dropped, any "upstream capacity" used for that packet was wasted !**</span>

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.3-6
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 