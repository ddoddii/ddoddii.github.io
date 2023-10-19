+++
author = "Soeun"
title = "[네트워크] P2P applications"
date = "2023-10-01"
description = "P2P application(BitTorrent) vs. Server-client architecture "
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
+++

## P2P application
P2P 는 peer-to-peer 이다. 항상 켜져있는 서버가 없고, 임의의 End system 끼리 직접적으로 소통한다. peer 들은 간헐적으로 연결되고, IP 주소가 바뀐다. 예시로는 file distribution(BitTorrent), Streaming, VoIP 가 있다. 

### File distribution : client-server vs P2P

우선 **client-server 시스템**부터 보자. 한대의 서버가 있고, 서버의 upload capacity 를  $u_s$ 라 하자. 파일 사이즈는 F bits 이고, N 대의 클라이언트가 파일을 다운받으려 한다. ith peer 의 upload capacity 는 $u_i$, ith peer 의 download capacity 는 $d_i$ 이다.  

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5f5b4cc0-5cf2-4295-8eac-3b85f0640c60)

이때 서버는 N개의 파일 복사본을 업로드 해야 한다. 한개의 파일을 전송하는데 필요한 시간은 $F/u_s$ 이고, N개의 파일을 전송하는데 필요한 시간은 $NF/u_s$ 이다. 

각 클라이언트는 파일의 복사본을 다운받아야 한다. $d_{min}$ 은 가장 작은 클라이언트의 다운로드 속도이다. 그러면 가장 오래 걸리는 클라이언트의 다운로드 시간은 $F/d_{min}$ 이 된다. 따라서 아래의 식으로 F bit의 파일을 N개의 클라이언트에게 나누어주는데 걸리는 시간의 최소값을 구할 수 있다. 아래의 시간은 N이 증가할수록 계속 증가한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/903b55cb-8483-446f-9027-bbc7a1185241)

P2P 구조에서 도 File distribution time 을 분석해보자. 초기에는, 서버 역할을 하는 peer 만 파일을 가지고 있으므로, 이때 업로드하는데 걸리는 시간은 $F/u_s$ 로 같다. 가장 오래 걸리는 클라이언트의 다운로드 시간은 똑같이 $F/d_{min}$ 이다. 

하지만 달라지는 점은, 클라이언트가 다시 파일을 자기들끼리 재분배할 수 있다는 것이다. 시스템 전체에서 봤을 때, 총 upload capacity 는 서버 + 각 peer 의 upload rate 이다. 즉, $u_{total} = u_{s}+ u_{1} + ... + u_N$ 이다. 시스템 내부에서 F bits 를 N peers 에게 전달해야 하니, 총 NF bits 를 전달한다. 따라서 최소한의 분배 시간은 $NF/(u_s+u_1+...+u_N)$ 이다. 

따라서 P2P 에서 파일을 분배하는데 걸리는 최소한의 시간은 아래와 같이 구할 수 있다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/69999e2d-59ad-4d86-822f-db8183522851)

**Client-server vs. P2P**

아래 표에서 볼 수 있듯이, 사용자가 늘어나면 P2P 구조가 더 이득이다. 즉, P2P 구조는 scalability 가 좋다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/beb8ffd2-557f-47e5-a912-adc8214e283d)

### P2P file distribution : BitTorrent

BitTorrent 는 파일 분배를 위한 프로토콜이다. peer 들이 파일을 분배하기 위해 그룹에 참여하는데, 이것을 **torrent** 라고 한다. torrent 내에 있는 peer 들은 file chunk 들을 보내고, 받는다.  **tracker** 를 가지고, torrent 에 참여하는 peer 들을 트래킹한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/25c489af-4df7-422a-9566-316fb757b2dc)

새로운 Peer 가 torrent 에 들어왔다고 하자. 새로운 peer 는 아직 파일 청크가 없지만, 점차 다른 peer 에게 받을 것이다. tracker 는 이 새로운 peer 에게 랜덤으로 선택된 peer 의 리스트를 준다 (랜덤으로 50개를 선택했다고 하자). 새로운 peer 는 이 리스트에 있는 peer 들과 TCP 연결을 하려고 한다. TCP 연결에 성공한 peer 들을 'neighboring peer' 라고 하자. 이 neighboring peer 는 고정된 것이 아니라, 시간이 지나면서 계속 바뀔 것이다. 

각 Peer 는 자신이 서버/클라이언트 역할을 동시에 한다. 그리고 각 peer 는 자신이 누구와 청크를 교환할 지 계속 바꿀 수 있다. peer 가 계속 들어왔다 나갈 수 있는데, 이것을 **churn** 이라고 한다. 

각 peer 는 파일 청크의 조각들을 가지고 있을 것이다. 새로운 peer 는 neighboring peer 에게 그들이 가지고 있는 청크들을 달라고 할 것이다. 만약 L 개의 이웃이 있다면, L 개의 청크를 얻게 될 것이다. 이 지식을 가지고, 자신이 없는 청크를 찾아나갈 것이다. peer 가 파일의 전체를 갖게 되면, torrent 를 나갈 수도 있고, 아니면 계속 있을 수 있다. 

#### BitTorrent : requesting, sending, file chunks

여기서 2가지 질문이 생긴다. 1) 새로운 peer 는 이웃 중 누구에게 청크를 요청해야할지 ? 2) 요청받은 청크를 이웃 중 누구에게 보내야할지 ?

**requesting chunks**

각 시간마다, 다른 피어들은 청크의 다른 조각들을 가지고 있다. 새로운 peer 는 각 peer 에게 그들이 가지고 있는 청크를 물어본다. 이때, 가장 희귀한 것 (rarest) 부터 물어본다. 

**sending chunks : tit-for-tat**

새로운 Peer 는 자신에게 청크를 가장 빠른 rate 로 보내는 상위 4명의 peer 에게 청크를 보낸다. 그리고 10초마다 top 4 를 갱신한다. 이 top4 를 unchoked 되었다고 하고, 나머지 다른 peer 들은 choked 되었다고 한다. 

30초마다는 랜덤으로 다른 peer 를 선택한다. 왜냐하면 , 또 새롭게 torrent 를 조인하는 사용자도 파일 청크를 줘야 하기 때문이다. 