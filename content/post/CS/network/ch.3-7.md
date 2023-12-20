+++
author = "Soeun"
title = "[네트워크] TCP Congestion Control"
date = "2023-10-14"
summary = "TCP가 congestion control 을 하는 방법들"
categories = [
    "CS"
]
tags = [
    "network"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "how-TCP-controls-congestion"
series = ["Network"]
series_order = 12
+++
{{< katex >}}

## TCP congestion control

TCP 의 중요한 요소 중 하나는 congestion-control mechanism 이다. TCP 는 end-to-end congestion control 을 가지고 있어야 한다. 왜냐하면 IP layer 는 네트워크 congestion 에 대해 피드백을 제공하지 않기 때문이다. 

TCP 의 첫번쨰 접근 방법은, 각 sender 가 보내는 속도를 네트워크의 congestion 정도에 따라 조절하는 것이다. 하지만 이 방법은 3가지 질문을 불러온다. (1) TCP sender 가 전송하는 속도를 어떻게 제한할 것인지 ? (2) TCP sender 는 보내는 길에 congestion 이 발생했다는 것을 어떻게 알 것인지? (3) end-to-end congestion 에 따라 전송 속도를 바꿀 때, 어떤 알고리즘을 사용해야 하는지? 

첫번째로, **TCP sender 가 연결로 데이터를 보내는 속도를 제한하는 방법**을 보자. 저번 장에서, TCP 연결은 receive buffer, send buffer, 각종 변수들 (rwnd, LastByteRead .. ) 가 있다고 했다. TCP sender 는 congestion window(cwnd) 라는 또 다른 변수를 도입한다. 이것은 TCP sender 가 데이터를 보내는 속도를 제한한다. sender 가 가지고 있는 unacked data 는 양은 , min(cwnd, rwnd) 값을 넘어서는 안된다. 

$$LastByteSent - LastByteAcked \leq min(cwnd,rwnd)$$ 

여기서는 cwnd 에 집중하기 위해 receiver 쪽의 버퍼는 매우 크다고 하자. 그러면 sender 쪽의 unacked data 는 cwnd 에 의해 결정된다. 대략적으로, RTT 가 시작할 때 sender 가 cwnd bytes 를 커넥션으로 보내고, RTT 가 끝날 때 ACK 를 받는다고 하면, sender의 sending rate 는 대략적으로 \\(cwnd/RTT\ bytes/sec\\) 이다. 

두번째로, **TCP sender 가 자신과 목적지 사이에 congestion 이 있는지 감지하는 방법**을 보자. "loss event" 를 timeout 또는 three duplicate ACKs 로 정의하자. congestion 이 발생하면, 경로 상에 하나 이상의 라우터 버퍼가 overflow 된 것이고, 데이터그램이 드랍된 것이다. 이 드랍된 데이터그램은 sender 에게 loss event 를 발생시킨다. 따라서, sender 는 경로 상에 congestion 이 발생했다고 인식한다. 

그럼, 네트워크가 congestion-free 일 때를 보자. TCP sender 에게 ACK 가 모두 정상적으로 도착할 것이다. TCP 는 이 ACK 를 받고, 모든 것이 잘 도착했구나 ! 생각하고, congestion window size(cwnd) 를 증가시킨다. ACK 가 느린 속도로 도착하면, cwnd 를 증가시키는 속도도 느릴 것이다. TCP 가 ACK 를 cwnd 를 증가시키는 트리거로 사용하기 때문에, TCP 는 **self-clocking** 이라고도 부른다. 

그러면 가장 중요한 질문이 남는다. **TCP sender 가 보내는 속도를 어떻게 조절할 것인가?**   
TCP 는 주어진 bandwidth 는 다 쓰면서, 패킷을 잃어버리지 않아야 한다. 이 최적의 전송 속도를 어떻게 찾을 것인가? 접근 방식은 loss 가 발생할 때까지, 전송 속도를 증가시키는 것이다. 이것을 AIMD 라고 한다. 
- additive increase : loss 가 발생할 때까지 cwnd 를 1 MSS 만큼 증가시킨다. 
- multiplicative decrease : loss 가 발생하면, cwnd 를 절반으로 줄인다. 

![스크린샷 2023-10-22 오전 12 30 24](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7100d7a7-d4ee-4727-a7ad-0c962d44ff3e)

cwnd 는 dynamic 하고, network congestion 에 의존하는 함수이다. 

### TCP Slow Start

TCP 연결이 시작되면, cwnd 의 초기값은 1MSS 이다. 그래서 초기 sending rate 는 MSS/RTT 이다. 만약 MSS = 500byte 이고, RTT = 200msec 이면, 초기 전송 속도는 고작 20kbps 밖에 안된다. 하지만 주어진 bandwidth 는 이것보다 훨씬 클 수 있다. 따라서, 이 slow-start 상태에서는 cwnd 가 1MSS 에서 시작해서 ACK 를 받을 때마다 1MSS 씩 증가한다.  하나의 segment 마다 1MSS 씩 증가하므로, 결과적으로 더블링하는 것과 같다. 그래서 지수적으로 빠르게 증가한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2177dd05-1a3a-44b2-aa37-9e89d4c35fbf)

**이 지수적으로 증가하는 것이 언제 끝나야 할까?** 

첫번째로,  **timeout 에 의한 loss event** 가 생기면, TCP 는 cwnd 를 다시 1 MSS 로 설정하고, 새로운 slow start 프로세스를 시작한다. 또한, slow start threshold(ssthresh) 를 loss가 발생한 때의 cwnd 값의 절반으로 설정한다. 이 ssthresh 후에도 지수적으로 증가시키는 것은 조금 위험하므로, ssthresh 를 지나면 cwnd 를 linearly 하게 증가시킨다. 이 상태를 **congestion avoidance mode** 라고 한다. 

두번째로, **3 duplicate ACK 에 의한 loss event** 가 생기면 TCP 는 fast retransmit 를 하고 fast recovery state 에 들어간다. dup ACK 는 네트워크가 Segment 를 전달할 수 있는 여력이 있다는 의미이다. 이 때도 cwnd 값은 반으로 줄고, 그 후에는 선형적으로 증가한다. 이것을 **TCP Reno** 라고 한다. 

**TCP Tahoe** 는 cwnd 를 언제나 1로 다시 설정하는 것이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/dd634768-add9-4c37-8c10-07ae51470004)


### Congestion Avoidance

CA 상태에서는, cwnd 는 선형적으로 증가한다. 즉 RTT마다 두배를 하는 대신,  RTT 마다 1 MSS 씩 증가한다. 이것을 구현하는 방법은, TCP sender 가 ACK 가 올 때마다 cwnd 를 MSS bytes (MSS/cwnd) 씩 증가시키는 것이다. 만약, MSS 가 1460bytes 이고 cwnd 가 14600bytes 라면, RTT 내에 10개의 segment 가 보내진다. 각 ACK는 cwnd 의 사이즈를 1/10 MSS 씩 증가시키고, 10개의 ACK 가 도착한 후엔, cwnd가 1MSS 만큼 증가했을 것이다.  이것을 식으로 나타내면, ACK 가 올 때마다 \\(cwnd=cwnd+MSS*(MSS/cwnd)\\) 가 된다. 

**이 선형적으로 증가하는 것은 언제 끝나야 할까?** 

첫번째로, timeout 시에는, cwnd = 1MSS 가 되고, ssthresh = cwnd/2 가 된다. 

두번째로, triple duplicate ACK 시에는, 네트워크가 여전히 segment 를 전송할 수 있다고 생각한다. TCP 는 cwnd 를 절반으로 줄이고, +3 MSS 를 더한다. ssthresh 는 loss 가 발생한 때의 cwnd 절반 값으로 설정한다. 그 다음 fast-recovery state 에 들어간다. 

### Fast Recovery

fast recovery 상태에서는 duplicate ACK 가 올 때 마다 1 MSS 씩 증가시킨다. 그리고 아까 잃어버린 패킷에 대한 ACK 가 오면, 다시 congestion avoidance 상태로 들어간다. 

만약 아까 재전송한 패킷에 대해 timeout 이 발생하면, slow start 상태로 간다. cwnd=1MSS 로 설정하고, ssthresh 는 loss 가 발생했을 때의 cwnd 값의 절반으로 설정한다. 

**TCP Tahoe** 는 timeout 이든, triple dup ACK 던 cwnd = 1MSS 로 설정한다. 반면, 더 새로운 버전인 **TCP Reno** 는 fast recovery 를 도입했다. 

아래 TCP congestion control 의 FSM 을 찬찬히 보면 더 이해가 잘 된다 ! 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/10f8fdea-0b7c-44cb-a0ac-694d2b0a989d)

### TCP throughput

TCP 연결의 throughput의 평균은 어떻게 될까? 여기서는 slow-start 단계들을 무시할 것이다. 왕복 인터벌에서, TCP 가 데이터를 보내는 속도는 congestion window 와 현재 RTT 에 의해 결정된다. window size = w bytes,  round trip time = RTT sec 일 때, TCP 의 전송 속도는 대략적으로 w/RTT 이다. 여기서 loss가 발생하기 전까지, w 를 1 MSS 만큼 증가시킨다. 그래서 전송 속도는 w/(2*RTT) ~ W/RTT 까지 이다. 

 따라서,  \\(avg \ TCP \ thruput=\frac{3}{4}\frac{W}{RTT}bytes/sec\\) 이다.  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/df41cf61-1fda-4bf4-98a2-0a5cf3bffccd)

**TCP futures : TCP Over "long, fat pipes"**

Loss probability(L) 을 고려한 \\(TCP \ throughput = \frac{1.22*MSS}{RTT\sqrt{L}}\\) 이다. 


### TCP Fairness
fairness goal : K TCP session 이 bandwidth R 을 가진 같은 bottleneck link 을 공유하면, 각각 세션은 평균 속도의 R/K 를 가져야 한다. 

TCP 는 fair 한데, 아래 그래프를 보면 알 수 있다. 만약 연결1,2 가 둘다 full bandwidth 아래면, cwnd 를 늘릴 것이다. 그러다가 B 지점에서 loss 가 발생하면 둘 다 cwnd 를 줄일 것이다. 그러다가 점차 equal bandwidth 를 공유하는 지점으로 수렴한다. 

<img width="387" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bf9ab12d-eecb-4f3c-a526-fd475942ab42">

**Fairness and UDP**

요즘 많은 멀티미디어 어플리케이션은 이 congestion control 기능이 없는 UDP 를 사용한다. 동영상 같은 것은 congestion control 을 하며 전송 속도를 느리게 하는 것보다, 빠르게 일정한 속도로 전송하되 패킷 loss 는 감수하는 방식이다. 그래서 UDP 는 TCP 관점에서는 Fair 하지 않다. 왜냐하면 네트워크의 congestion 은 고려하지 않은 채, 일정한 속도로 데이터를 전송하기 때문이다. 


### Explicit Congestion Notification(ECN) : Network-assisted Congestion Control

네트워크 에서 congestion 신호를 TCP 에게 주는 방식도 있다. 이것을 ECN 이라고 한다. 네트워크 라우터가 IP header 의 2 bits 을 이용해 congestion 을 표시한다. 이 congestion indication 은 receiving host 에게 전달된다. receiver 는 ACK segment 에 ECE bit 을 통해 sender 에게 congestion 정보를 전달해준다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition, ch.3-7
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 