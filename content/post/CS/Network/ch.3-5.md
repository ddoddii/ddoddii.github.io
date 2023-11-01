+++
author = "Soeun"
title = "[네트워크] connection-oriented transport : TCP"
date = "2023-10-11"
description = "TCP 에 대한 모든 것"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "Everything-about-TCP"
+++

## connection-oriented transport : TCP

### 1. The TCP Connection
TCP 는 connection-oriented 이다. 하나의 어플리케이션 프로세스가 데이터를 전송하기 전에, 두 프로세스는 서로와 연결하는 'handshake' 과정을 거쳐야 한다. TCP 는 point-to-point 로, 하나의 sender 와 하나의 receiver 만 있다. TCP 는 전송 계층의 프로토콜이므로, 네트워크 계층에서 일어나는 것이 아니라 end system 에서 돌아간다. TCP 는 full-duplex service 를 제공한다. 즉, 하나의 연결을 가지고 데이터를 양방향으로 전송할 수 있다. 

이제 TCP 연결이 어떻게 되는지 보자. 하나의 호스트에서 돌아가는 프로세스가 다른 호스트의 프로세스와 연결을 하고 싶다고 하자. 연결을 시작하는 쪽을 클라이언트, 받는 쪽을 서버라고 한다. 클라이언트는 서버에게 TCP segment 를 전송하고, 서버도 클라이언트에게 특별한 TCP segment 를 보내며 응답한다. 마지막으로 클라이언트는 세번째 TCP segment 로 응답한다. 처음 2번의 segment 는 어떠한 payload (application-layer data) 를 가지고 있지 않다. 세번째 segment 는 payload 를 가지고 있을 수 있다. 총 3번의 과정이 있어서 '**three-way handshake**' 라고도 불린다. 

TCP 연결이 수립되면, 두개의 어플리케이션 프로세스는 데이터를 전송할 수 있다. 
클라이언트가 데이터를 전송할 때, 소켓을 통해 데이터를 보낸다. 데이터가 소켓을 통과하면, 이제 클라이언트의 TCP 의 손으로 넘겨진다. TCP 는 이 데이터를 connection 의 send buffer 로 보낸다. 그리고, TCP 는 이 send buffer 에서 데이터의 청크들을 잡아서 네트워크 레이어로 보낸다. segment 에 있을 수 있는 최대한의 데이터 양은 **maximum segment size(MSS)** 로 제한된다. MSS 는 맨 처음에 가장 큰 link-layer frame (이것은 maximum transmission unit, MTU 라고도 불린다) 에 의해 정해진다.  TCP segment  + TCP/IP header 가 single link-layer frame 안에 들어가도록 MSS를 세팅한다. 헷갈리지 말아야 할 점은, MSS 는 오직 어플리케이션 계층의 데이터가 segment 안에 들어가는 양을 표시하는 것이고, header size 는 별도이다 ! 

TCP 는 클라이언트 데이터를 TCP header 과 페어링해서, TCP segment 를 생성한다. 이 segment 들이 네트워크 계층으로 전송되어서, 네트워크 계층 IP datagram 으로 캡슐화된다. 이 IP datagram 이 네트워크로 보내진다. 받는 쪽에서 segment 를 받으면, 이 segment 의 데이터는 TCP connection's receive buffer 에 저장된다. 그러면 받는 쪽의 어플리케이션은 여기서 데이터를 읽어온다. 

TCP connection 은 buffers, variables, socket connection 으로 이루어지는 것을 볼 수 있다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8595f793-ab1a-4f2a-ae03-4c91a0ba2b60">

### 2. TCP Segment Structure

TCP segment 의 구조에 대해 알아보자. TCP segment 는 header field 와 data field 로 구성된다. MSS 는 segment 의 data field 사이즈를 제한한다. 

아래는 TCP segment 의 구조이다. UDP 와 마찬가지로, header 는 source port # 와 destination port # 포함한다. 그리고 checksum field 가 있다. 

추가적으로, 있는 것은, 아래이다. 
- 32-bit sequence number field
- 16-bit receive window
- 4-bit header length field
- options field
- flag field : ACK bit, (RST,SYN,FIN), PSH, URG


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2359a6f1-a8d0-49c6-95c5-edd46f30eb00)


<span style="font-size:110%"><span style="background-color: #EBFFDA">**Sequence Numbers**</span></span>  

TCP 는 데이터를 unstructured, ordered, stream of bytes 로 본다. segment 의 **Sequence number 는 segment data 안에 있는 첫번째 byte 의 byte stream "number" 이다.** 만약 HostA 가 HostB 에게 500,000 bytes 의 파일을 보낸다고 하자. 이 때 MSS = 1000byte 이면, 아래와 같이 파일을 자를 것이다. 그러면 각 segment 의 sequence # 는 0, 1000, 2000 ,,, 이렇게 될 것이다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c0a970a6-89ee-4543-894a-2eeca6163362">

<span style="font-size:110%"><span style="background-color: #EBFFDA">**Acknowledgment Numbers**</span></span>

Acknowledgment number 는 상대방이 보내기를 기대하는 next byte 의 seq. number 이다. 만약 HostA 가 B 로 부터 0~535bytes 데이터를 받았고, B에게 segment 를 전송할 차례라고 하자. HostA 는 B 로부터 다음 byte인 536 byte 가 오기를 기대하고 있다. 그래서 HostA 는 ACK 에 536 을 넣어서 보낸다. 

다른 예시로는, 만약 똑같이 A가 B로부터 0~535bytes 를 받고, 900~1000까지의 byte 를 받았다고 하자. A는 어떤 이유로 536~999 byte 는 받지 못했다. 이때, A는 아직도 536byte 데이터가 오기를 기다리고 있다. 그래서, A는 여기서도 ACK 에 536을 넣어서 보낸다. 이러한 TCP 의 특성은 **cumulative acknowledgments** 라고 한다. 

세번째 예시로는 , segment 의 byte 순서가 맞지 않은 채로 받은 것이다. 첫번째로 0~535byte , 두번째로 900~1000 byte, 세번째로 536~999byte 를 받았을 때, HostA 는 이 순서가 맞지 않는 segment 를 어떻게 처리해야 할까? 이것은 RFC 측에서는 명시하지 않고, TCP 를 구현하는 프로그래머들에게 맡긴다. (1) out-of-order segment 를 버리는 방법과, (2) out-of-order bytes 를 보관하고, gap 을 채울 missing byte 를 기다리는 방법 2가지가 있다. 

아래 사진을 보면 seq.# 와 ack.# 를 이해할 수 있다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f16d7827-bd2f-4138-9e54-84658a56d100)

![스크린샷 2023-10-21 오후 3 10 27](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/28f03287-1bb0-4e86-9852-d9de76bde42e)

### 3. Round-Trip Time Estimation and Timeout

rdt protocol 과 비슷하게, TCP 도 잃어버린 segment 를 복구하기 위해 timeout/retransmit 메커니즘을 이용한다.

**그렇다면 timeout interval 의 길이를 어떻게 설정해야 할까?** 우선, RTT 보다는 길게 설정해야 한다. 하지만 RTT 는 네트워크 상황 등에 따라 변동한다. timeout 이 너무 짧으면, 불필요한 재전송을 야기할 수 있다. timeout 이 너무 길면, segment loss 에 대한 반응이 너무 느려진다. 

**RTT는 어떻게 측정해야 할까?** **SampleRTT** 는 segment 전송 ~ ACK 받을 때 까지 시간이다. 이 SampleRTT 는 라우터의 congestion, end-to-end 사이의 길이 등 변수들에 의해  segment 에 따라서 달라진다. 그래서 우리는 더 "smooth" 한 **EstimatedRTT** 를 사용한다. 

$EstimateRTT = (1-\alpha)*EstimateRTT+\alpha * SampleRTT$

이것은 exponential weighted moving average 로, 과거 샘플들의 중요도는 지수적으로 감소하고, 최근의 샘플들에 대한 중요도는 올라간다. SampleRTT 보다 EstimatedRTT 가 훨씬 변동성이 적은 것을 볼 수 있다. $\alpha$ 값은 주로 0.125 를 사용한다.

![스크린샷 2023-10-21 오후 3 19 44](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1f32e640-3350-4414-a6f3-e3fd0869f34e)

그러면 timeout interval 을 EstimateRTT + 'safety margin' 으로 설정하면 된다. EstimateRTT 에 변동성이 크면, 더 큰 safety margin 을 설정해주면 된다. 그래서, 이 변동성(DevRTT)을 측정해서 마진으로 넣어준다. 

$DevRTT = (1-\beta)*DevRTT+\beta *|SampleRTT-EstimateRTT|$

따라서, 최종적인 $TimeoutInterval = EstimatedRTT + 4*DevRTT$  로 설정한다. 


### 4. Reliable data transfer
네트워크 계층의 서비스(IP service)는 신뢰할 수 없다. IP 는 datagram delivery 를 보증하지 않는다. 따라서, TCP 는 reliable data transfer service 를 제공하려고 노력한다. 우선, TCP sender 쪽에서 어떤 일이 일어나는지 보자.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**TCP Sender Events**</span></span>

3가지 이벤트가 있다 : data received from applicatoin, timer timeout, ACK receipt

1. data rcvd from app

	seq.# 를 가지고 segment 를 만든다. 돌아가고 있는 타이머가 없으면, 타이머를 시작한다. 타이머는 가장 오래된 unacked segment 와 관련있다고 생각하면 된다. 타이머가 끝나는 시간은 위에 했던 것처럼 EstimatedRTT 와 DevRTT 를 가지고 계산한다. 

2. timeout

	timeout이 발생하면, 아직 ack 가 안 온 segment 중 가장 작은 seq. # 를 재전송하고, 타이머를 다시 시작한다. 

3. receiver 로부터 ACK 도착

	ACK 가 도착하면, TCP 는 이 ACK 값을 SendBase 의 값과 비교한다. SendBase 는 가장 오래된 unacked byte 이다. (SendBase-1 값은 수신자 쪽에서 발신자가 정확히 받았다고 알고 있는 last byte의 seq.# 이다.) TCP 는 cumulative acknowledgment 를 사용하므로, SendBase 이전의 byte 들은 ack 를 받은 byte 들이다. 받은 ACK 값 > SendBase 이면, 현재 unack 상태에 있는 segment 에 대한 ACK 가 온 것이므로, SendBase 를 업데이트한다. 그리고 아직도 unack 된 segment 가 있으면, 타이머를 재시작한다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**TCP Sender FSM**</span></span>

![스크린샷 2023-10-21 오후 3 38 55](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/09460c9c-1d33-4e9b-83aa-2507ffeee530)

<span style="font-size:110%"><span style="background-color: #EBFFDA">**TCP : retransmission Scenarios**</span></span>

3가지 시나리오(lost ACK, premature timeout, cumulative ACK) 를 보자.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9d4983f7-38f3-4da4-bd5f-8fe46a79ddf3)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/682f591c-6547-4666-9da9-690888ffdbd1)



<span style="font-size:110%"><span style="background-color: #EBFFDA">**TCP fast retransmit**</span></span>

timeout 에 의해 trigger 되는 retransmission의 문제점은, time-out 시간이 길 수 있다. 그래서 잃어버린 패킷을 전송하는데 긴 지연이 발생할 수 있다. 따라서, 잃어버린 segment 를 duplicate ACKs 로 감지하는 방법이 있다. duplicate ACK 는 sender 가 이미 받은 ACK 가 중복되는 것이다. 

우선, receiver 의 ACK generation policy 부터 보자. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7cd551d7-e0b0-4cf2-9522-43fe9533202c)

sender 가 같은 데이터에 대해서 3 ACK 를 받으면, 진짜 그 데이터를 잃어버린 것이라고 판단하고, 재전송한다. 이것을 triple duplicate ACKs 라고 한다. 왜 2개가 아니라 3개냐? 라고 하면, 우선 sender 쪽에서는 연속적으로 데이터를 계속 보낸다. receiver 쪽에서는 이 패킷의 순서가 바뀌어서 올 수 도 있다. 그래서 duplicate ACK 를 보낼 수 있는데, 2개가 아니라 3개의 ACK 가 오면 진짜 패킷을 잃어버린 것이라고 판단할 수 있다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/227288ff-0422-4625-9cfa-12b3fcd00568)

앞서서, 잃어버린 패킷들에 대해 대응하는 방식에는 Go-Back-N , SR 프로토콜이 있다고 했는데, TCP 는 어디에 해당할까?

TCP sender 는 오로지 전송됨 & Unacked byte (=SendBase) 와, 다음 전송할 seq.# (=NextseqNum) 만 저장한다. 이렇게 봐서는, TCP 는 GBN 과 비슷해 보인다. 하지만, 가장 큰 다른 점은 TCP 는 out-of-order segment 를 GBN 처럼 버리지 않고 buffer 에 저장한다. 또, sender 가 1,2,...,N 의 segment 를 보내고, 이 중 n segment 가 lost 되었다고 하자. 이것을 제외하고 N-1 개의 segment 에 대한 ACK 가 왔다고 했을 때, GBN 은 이 n segment 뿐만 아니라 이 뒤의 모든 segment 도 재전송한다. 하지만 TCP 는 n segment 만 재전송한다. 또한, TCP 는 cumulative ACK 에 의해 만약 n segment 에 대한 timeout 전에 n+1 ACK 가 도착하면, n segment 를 재전송하지 않는다. 

그래서 TCP 에 대해 selective acknowledgment 가 도입되었다. 이것은 TCP receiver 가 out-of-order segment 를 선별해서 ACK 를 보내는 것을 허용한다. 

### 5. Flow control

TCP connection 의 각 host 는 Buffer 를 가지고 있다고 했다. 만약 receiver 의 어플리케이션이 buffer에서 데이터를 읽어오는 속도보다, sender 가 receiver 에게 데이터를 보내서 buffer 를 채우는 속도가 더 빠르면 overflow 가 발생할 수 있다. 

![스크린샷 2023-10-21 오후 4 04 55](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/91207e83-2ad2-4e06-a3a9-0ffc2f2d7bec)

TCP 는 **flow-control service** 를 제공한다. receiver 는 sender 를 컨트롤해서, sender 가 데이터를 너무 빨리 전송해서 receiver 쪽의 buffer 를 overflow 시키는 일이 없도록 한다. receiver 는 자신이 가진 free buffer space 를 TCP header 에 rwnd(receiver window) 값을 통해 계속 알려준다. HostA 가 HostB 에게 대용량의 파일을 전송한다고 하자. HostB 는 이 연결에 버퍼를 할당하고, 이 사이즈를 RcvBuffer 로 설정한다. 그리고 HostB 의 어플리케이션은 이 버퍼에서 데이터를 읽어오는데, 이 때 2가지 변수를 정의한다. 

- LastByteRead : 어플리케이션 프로세스가 버퍼에서 읽은 마지막 byte 의 number
- LastByteRcvd : 네트워크로부터 받아서 버퍼에 저장된 마지막 byte 의 number
  
LastByteRcvd - LastByteRead <= RcvBuffer 이어야 한다.

따라서, $rwnd = RcvBuffer - [LastByteRcvd - LastByteRead]$ 로 설정한다. 

그러면, sender 는 자신의 window size 를 receiver 의 rwnd 값으로 제한한다. 이렇게 하면, receiver buffer 가 overflow 하는 것을 막을 수 있다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/371fb925-2dd6-4549-937a-8b8e90b29201)

만약, rwnd = 0 이 되면 어떻게 할까? 이 때는 sender 쪽에 얘기해서 더 이상 데이터를 안 보내게 한다. 하지만 이 경우에, receiver 쪽에서 보낼 데이터가 없으면 버퍼가 비어도 sender 에게 알려줄 길이 없다. 따라서 버퍼가 다 차도, rwnd = 1 byte 로 설정해둔다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.3-5
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안
