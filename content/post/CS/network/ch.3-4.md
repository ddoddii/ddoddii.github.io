+++
author = "Soeun"
title = "[네트워크] Principles of reliable data transfer "
date = "2023-10-09"
summary = "rdt 의 발전 과정과 GBN, SR"
categories = [
    "CS"
]
tags = [
    "network"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "RDT-and-GBN-SR"
series = ["Network"]
series_order = 9
+++
{{< katex >}}

## principles of reliable data transfer

신뢰감 있는 데이터 통신을 하는 것은 전송 계층 뿐만 아니라 링크 계층, 어플리케이션 계층도 상관 있다. 궁극적으로 신뢰 있는 데이터 통신을 하려면, 전송 계층에서 데이터를 보낼 때 (즉 아래 모든 계층들을 통과할 때) '신뢰할만한 채널' 을 통과해야 한다. '신뢰할만한 채널'아래 모든 계층들을 추상화한 서비스이다. 

하지만 전송 계층 아래 있는 계층들에서 오류가 나지 않는다는 보장이 없다. 따라서 우리는 이 아래 채널들을 '신뢰할 수 없는 채널' 이라고 생각하자. 따라서 아래 그림에서 (a) 는 전송 계층에서 실제로 제공해야 하는 서비스이고, (b) 는 실제 현실세계를 반영한 것이다. 

<img width="511" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7b1db62d-34a3-4fa2-bf13-e83d2c0d9fc7">

이 '신뢰할 수 없는 채널' 이 reliable data transfer protocol(rdt) 의 복잡도를 결정한다. 

### Reliable Data Transfer(RDT)

우선 rdt 에 대해 알아보자.  전송 계층에서는 아래와 같은 일들이 일어난다. 
<img width="554" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/01e4d55e-f467-4c16-9c29-087ea3feef0e">
rdt 에서 sender, receiver 역할을 하는 것은 finite state machines(FSM) 이라고 가정한다. FSM은 state 가 있는데, 이 다음 state 는 next event 에 의해 결정된다. 

<img width="545" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0726ea82-f869-4b53-99da-78b7c62aa68b">

### rdt 1.0 : reliable transfer over a reliable channel

rdt 1.0 은 전송 계층 아래에 있는 채널들이 완벽하게 신뢰할 수 있다고 가정한다. 즉 bit error 과 packet loss 가 없다. sender, receiver 별로 각각 FSM 이 있다. sender 는 아래 채널로 데이터를 전송하고, receiver 는 아래 채널로부터 데이터를 받는다. 

**rdt 1.0 FSMs**

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/514211b2-b6b6-4a96-a946-308104d944e3">

### rdt 2.0 : channel with bit errors

rdt 2.0 은 아래 채널에 bit error 가 있다고 가정한다. 이 bit error 들은 패킷들이 전송되고, 버퍼될 때 네트워크의 물리적인 구성요소에서 발생할 수 있다. 그리고 아직까지는 보내는 순서대로 패킷을 받는다고 하자. 

우선 사람들이 의사소통 오류가 났을 때 어떻게 행동하는지 생각해보자. 제대로 알아들었으면 "OK" 라고 할 것이고, 못 알아들었으면 다시 말해달라고 할 것이다. 프로토콜도 마찬가지로 positive acknowledgments(ACKs) 와 negative acknowledgments (NAKs) 를 모두 사용한다. 이 메세지들은 수신자가 발신자가 보낸 메세지를 제대로 오류 없이 받았는지 알 수 있게 해준다. 이것을 ARQ(Automatic Repeat reQuest) protocol 이라고 한다. 

bit error 를 다루기 위해서는 3가지 기능이 더 필요하다. 

- Error detection
  
	UDP 에서 checksum 기능을 사용해서 bit error 를 탐지한다고 했다. rdt 2.0 에서도 checksum 을 사용한다. 
- Receiver feedback 
  
	수신자와 발신자 간의 소통으로는 ACK, NAK 를 사용한다. 
- Retransmission
  
	에러가 있는 패킷은 재전송해야 한다. 

**rdt 2.0 FSMs**

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3575b8dc-cc89-4918-81dd-40016d45969c">

**rdt 2.0 의 치명적인 약점**

rdt 2.0 은 ACK, NAK 에 굉장히 의존한다. 하지만 이 ACK, NAK 가 corrupt 된다면 ? 발신자는 수신자가 제대로 받았는지 알 수 없다. 그렇다고 다시 데이터를 재전송 할 수 도 없다. 이것은 duplicate 문제를 일으킬 수 있다. 왜냐면 수신자는 새로 전송받는 데이터가 재전송된 중복 데이터인지, 아니면 새로운 데이터인지 모르기 때문이다. 

그러면 무조건 다시 보내고, duplicate 문제를 다루는 기능을 넣는다면 ? 이 시나리오 일 것이다. 발신자는 수신자에게서 ACK/ NAK 답장이 안 오면 무조건 데이터를 재전송한다. 발신자는 패킷에 sequence number 를 부착한다. 그러면 수신자는 중복 패킷을 버리면 된다. 

### rdt 2.1 : Having 0,1 states

rdt 2.1 에서는 0,1 두 가지 state 가 있다. 왜냐하면 프로토콜 상태는 지금 보내는 패킷 / 받는 패킷이 sequence number 0 또는 1 인지를 반영한다. 

rdt 2.1 에서 sender 의 FSM은 아래와 같다. 0,1 두 가지 상태가 있는 것을 볼 수 있다. 수신자가 corrupted packet 을 받으면, 수신자는 NAK 를 보낸다. 


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9cd489ef-2a81-40d9-9ea8-9745f4f4a346)


아래는 rdt 2.1 의 receiver 의 FSM 이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/14f87866-f492-491c-b29c-7ba9a0d32cbc)

여기서 생각해 볼 수 있는데, 만약 수신자가 NAK 를 쓰지 않고, 직전에 성공적으로 받은 패킷에 대해서 ACK 를 보낸다면, 발신자는 동일한 패킷에 대해 2번의 ACK 를 받을 것이다. 그러면 발신자는 그 다음 패킷이 잘못 전송되었다는 것을 알 수 있다. 이것은 rdt 2.2 에서 NAK 를 쓰지 않고 ACK 만 쓰는 구조이다. 

### rdt 2.2 : NAK-free protocol

rdt 2.2 는 rdt 2.1 과 동일한 기능이지만, ACK 만 사용한다. NAK 를 사용하는 대신, 직전에 성공적으로 받은 패킷에 대해 ACK 를 다시 보낸다. 수신자는 무조건 ACK 를 보내는 패킷에 대해 sequence number 를 붙여야 한다. 발신자에게 duplicate ACK 가 도착하면, 이것은 NAK 와 같은 의미가 된다. 그래서 발신자는 두번의 ACK가 온 다음 패킷을 재전송 한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/51a4549a-a3dd-4afd-b0ab-1f389119c457)

### rdt 3.0 : channels with errors and loss

새로운 가정을 해보자. 밑에 있는 채널들은 에러 뿐만 아니라 패킷을 잃어버릴 수 도 있다.  패킷을 잃어버린 것을 어떻게 탐지할지, 그리고 잃어버렸으면 어떻게 대처해야 하는지 알아보자. 여기서 후자는 checksum, seq .# , ACK, 재전송 으로 커버할 수 있지만, 전자는 탐지할 수 없다.  

여기에 대한 접근은 **타이머를 설정**하는 것이다. 발신자는 ACK 가 올 '적당한' 만큼의 시간 동안 기다린다. 만약 ACK가 시간 내에 안 온다면, 발신자는 패킷을 잃어버렸다고 생각하고 패킷을 재전송한다. 만약 패킷이 단순히 지연되서 시간 내 ACK가 안 온 것이라면, 재전송은 duplicate 이 되겠지만, seq. # 가 이 문제를 해결할 수 있다. 여기서도 수신자는 ACK 를 보내는 패킷의 번호를 명시해야 한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c6e8a507-5613-4af6-9ade-ac8f18884fc6)

아래는 rdt 3.0 이 작동하는 4가지 시나리오이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/db830392-fa54-4c68-9b8b-9f565e8694c5)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bbf6ec1d-94a1-4809-b51d-6332f821d606)

그러면 rdt 3.0 의 성능에 대해 살펴보자. 타이머가 있으므로 그만큼 지연이 많이 발생한다. 

만약 1Gbps link 가 있고, 15ms propagation delay, 8000 bit packet 이 있다고 하자. 

$$D_{trans} = L/R = \dfrac{8000bits}{(10^9bits/sec)} = 8ms$$

$$U_{sender} = \dfrac{L/R}{RTT + L/R} = \dfrac{0.008}{30.008} = 0.00027$$


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fc550256-6e5d-49dd-8410-832a40b2f64f)


즉 발신자는 0.00027% 만큼만 바쁘다. 즉, 발신자는 30.008 msec 에 고작 1000byte 만 보낼 수 있기 때문에, 267kbps throughput 을 가진다. 1Gbps 링크에 267kbps 라니, 자원의 낭비가 너무 심하다. 

### Pipelined protocols : GBN, SR

Pipelining : 발신자는 아직 대답을 받지 못한 패킷들을 다수 보내는 것을 허용하는 것이다. 

이 방식으로 진행하면, seq # 가 0,1 보다 더 많이 필요하다. 그리고 발신자 or / and 수신자에게 버퍼링이 필요하다. 

<img width="537" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0fbba8ea-d53b-4f8d-b1ad-451bf7613b3e">

Pipelining 을 하면 rdt 3.0 에서 문제가 되었던 성능을 어느정도 개선할 수 있다. 
만약 3-packet pipelining 을 하면, stop-and-wait 프로토콜 보다 성능을 3배 증가시킬 수 있다.  

Pipelined protocols 에는 2가지가 있다 : Go-back-N , Selective Repeat

#### Go-back-N
GBN 에서 발신자는 파이프라인에 N개의 unacked 패킷을 가질 수 있다. 수신자는 오로지 순서가 맞는 ack 를 보낸다. 만약 0,1 패킷을 받고 패킷2를 못받았으면 계속 패킷1 에 대한 ack 만 보낸다. 즉, gap 이 있으면 그 다다음 순서들에 대한 패킷들에 대해서는 ack 를 보내지 않는다. 수신자는 unack 된 가장 마지막 패킷에 대해 타이머가 있다. 그 타이머가 끝나면, 모든 unack 된 패킷을 다시 보낸다. 

**GBN : Sender**

발신자는 패킷 헤더에 k-bit 의 seq. # 를 붙인다. 윈도우 사이즈는 N 이다. 즉 파이프라인에 N 개 보다 많은 unack 패킷이 있으면 안된다. 
<img width="557" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/3a1d1822-f194-467d-8f21-520483f39998">
- send_base : 가장 오래된 unack 패킷 
- nextseqnum : 다음으로 보내야 할 패킷의 seqnum
- 윈도우 내에서 [0,base-1] 에 있는 패킷들은 전송되었고 ack 를 받은 것
- 윈도우 내에서 [base, nexseqnum -1] 에 있는 패킷들은 보냈지만 ack 를 받지 못한 것 
- 윈도우 내에서 [nexseqnum , base + N -1] 에 있는 패킷들은 어플리케이션에서 데이터가 도착하면 바로 보낼 예정인 패킷들 
- [base + N] 뒤에 있는 패킷들은 앞에 있는 unack 패킷에 대해 ack 가 와야 보낼 수 있는 것 

윈도우가 슬라이딩 하기 때문에, GBN 은 sliding-window protocol 이라고도 불린다. 음.. 애초에 N개로 윈도우 사이즈를 제한하는 이유가 뭘까 ? 이것은 flow-control 때문이다. (뒤에서 더 자세히 다루겠다.)

**GBN : sender FSM**

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/29d99f08-d7ff-4815-a3bf-ce8cdfe75c8a">

**GBN : receiver FSM**

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8b3d43ff-a243-4166-b3f7-a31bcd8ca5ee">

GBN 은 ACK-only 이다. 맞게 받은 패킷에 대해서만 순서에 맞게 ACK 를 보낸다. 패킷 0,1 받고 패킷2 를 못받으면 계속 패킷 1 에 대한 ACK 를 보낸다. 이것은 duplicate ACKs 를 발생할 수 있다. 패킷에 대해 순서가 맞지 않으면, 버퍼링 하지 않고 그냥 버린다. 그래서 **수신자는 버퍼링이 필요 없다.**

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7baeac31-6018-4193-89bc-4447f7efdefc">

그러나 GBN 은 하나의 패킷에 문제가 생기면, 그 뒤에 있는 패킷들도 모두 같이 재전송한다. 이것은 불필요한 작업일 수 있다. 그래서 Selective Repeat  수신자가 받지 못한 패킷만 재전송한다. 

#### Selective Repeat

SR 에서는 수신자가 성공적으로 받은 패킷에 대해 개별적으로 모두 ACK 를 보낸다. 수신자는 패킷의 순서를 유지하기 위해 **버퍼가 필요**하다. 만약 패킷 0,1 을 받고 패킷2 를 못 받았으면, 뒤에 온 패킷3,4,5 를 버퍼에 저장해놓고 있는다. 그 다음 다시 패킷2가 성공적으로 오면, 패킷2와 버퍼에 저장해놓은 패킷3,4,5 를 순서대로 어플리케이션 레이어로 보낸다. 

발신자는 ACK 를 받지 못한 패킷만 다시 보낸다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/303578e7-1d3a-4dc9-85c8-ee416fc2bb3f">
SR 과 같은 경우에는 발신자의 윈도우 내에는 ack 를 받은 패킷과 unack 패킷이 섞여 있다. 

**SR : sender** 
- data from above : 다음 seqnum 가 윈도우 안에 있다면, 패킷을 보낸다.
- timeout(n) : 패킷n에 대해 타임아웃이 발생하면 패킷n 을 재전송하고, 타이머를 다시 시작한다.
- ACK(n) in [sendbase, sendbase + N] : 패킷n 을 받은 것으로 처리, n 이 unack 패킷 중 가장 오래된 것이라면 윈도우 베이스를 다음 unack seq.# 으로 옮긴다. 

**SR : receiver**
- pkt n in [rcvbase, rcvbase + N - 1]
	- ACK(n) 을 보낸다.
	- n 의 순서가 맞지 않으면 버퍼에 저장한다.
	- n 의 순서가 맞으면 어플리케이션 으로 전달하고, 윈도우 베이스를 다음 받지 못한 패킷으로 옮긴다. 
- pkt n in [rcvbase - N , rcvbase - 1]
	- 이미 수신자가 받은 패킷이지만, 다시 ACK(n) 을 보내준다. 
- 그 외 경우 : 무시한다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4e2db5fa-39d0-4d50-96db-2d74afc9565c">

**SR : dilemma**

만약 seq # 가 0,1,2,3 이 있고, window size = 3 이라고 하자. 그럼 아래와 같은 2가지 상황이 일어날 수 있다. 

<img width="418" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/22eedb90-6f61-49a9-8457-6a2e5a7049ad">

(a) 의 경우 정상적으로 작동하는 경우이다. 하지만 (b) 의 경우, 만약 발신자가 ACK 를 모두 못 받았다면, 아까 보낸 pkt0 을 다시 보낸다. 그리고 수신자는 다음이 seq. # 0 을 받을 차례이므로, 똑같은 패킷인지 모른채 그 패킷을 받는다. 그럼 중복 패킷 문제가 발생한다. 

이것은 seq # 와 윈도우 사이즈가 같으면 생기는 문제이다. 그래서 보통은 seq # > window size * 2 로 잡는다.

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.3-4
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 