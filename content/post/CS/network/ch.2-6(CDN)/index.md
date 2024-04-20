+++
author = "Soeun"
title = "[네트워크] Video Streaming and Content distribution Network"
date = "2023-10-02"
summary = "비디오 스트리밍에 쓰이는 DASH 분산 네트워크인 CDN"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
slug = "CDN"
series = ["Network"]
series_order = 7
+++
{{< katex >}}

## Video streaming and content distribution networks(CDNs)

요즘 Internet bandwidth 의 가장 큰 소비자는 video traffic 이다. 여기서 챌린지는, scale 이다. 어떻게 하면 1billion 의 유저들에게 도달할 수 있을까? 또다른 어려움은 , 유저들 간의 환경이 모두 다르다는 것이다. 각 유저마다 다른 capability (bandwidth rich/poor, wired/mobile..) 를 가지고 있다. 

해결책은 distributed, application-level infrastructure 를 쓰는 것이다. 

### Multimedia : video
비디오는 일정한 rate 로 연속된 이미지들이 보여지는 것이다. 이미지는 pixel 들로 이루어져 있다. 각 pixel 는 bits 로 이루어져 있다. 이미지를 인코딩하는데 필요한 bit 의 수를 줄이기 위해서, 이미지 내부에서 & 이미지들 간 redundancy 를 이용한다. 

이미지 내부에서는 spatial coding 을 한다. 이것은 이미지 내에서 비슷한 색이 반복될 때, 이 색이 몇번 반복되는지 정보를 저장하는 것이다. temporal coding 은 i번째 프레임에서 i+1 번째 프레임이 될때, 모든 정보를 전송하는 것보다 이 전 프레임과 달라진 부분만 전송하는 것이다. 

### HTTP Streaming and DASH
#### HTTP Streaming

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c90c7f91-a732-4146-9c92-7e37bd48278e)

서버는 비디오를 저장하고 있다가, 사용자가 보고자 하면 스트리밍 해준다. HTTP 서버에 specific URL을 가진 파일로 저장하고 있다가, 사용자가 TCP connection 을 통해 그 URL 을 요청하는 GET request 를 보내면 서버는 사용자에게 비디오 파일을 보내준다. 

하지만 이 방식으로는, 모든 사용자가 환경과 상관 없이 같은 encoding 비디오를 받게 된다. 따라서 아래 새로운 타입의 HTTP streaming 서비스 (DASH)가 도입되었다. 


#### DASH(Dynamic, Adaptive Streaming over HTTP)

DASH 는 비디오를 다른 버전들로 인코딩한다. 서버는 비디오 파일을 각각 청크들로 나눈 후, 각 청크를 다른 encoding rate(다른 bit rate, 다른 퀄리티) 로 저장한다. 서버는 **manifest file** 을 갖고 있는데, 이것은 encoding rate 이 다른, 같은 청크에 대해서 URL 을 제공하는 것이다. 

클라이언트는 처음에 manifest file 을 요청하고, 다른 버전들의 정보에 대해 알게 된다. 그 후 HTTP GET request 에 특정 URL byte range 를 명시해서, 하나의 청크를 고른다. 청크를 다운로드 할 때도, 다음 청크를 고르기 위해 주기적으로 bandwidth 를 검사한다. 그래서 현재 주어진 bandwidth 에서 최대한 높은 coding rate 를 가진 청크를 고른다. 따라서 매번 요청할 때마다 bandwidth 에 따라 다른 coding rate 를 가진 청크를 받을 수 있다. 

DASH 는 클라이언트에게 선택권이 있다. 
- when to request chuck
- what encoding rate to request
- where to request chunk (클라이언트에게 가까운 URL 서버에 요청을 보낸다)

### Content distribution networks(CDN)

또 다른 챌린지는, 동시에 수천명의 사용자에게 어떻게 컨텐츠를 스트리밍할지 이다. 

**option1 : single , large 'mega-server'**

하나의 거대한 서버를 둘 수 있지만 , 이것은 단점이 엄청나게 많다. 하나의 서버란 곧 single point of failure 를 뜻한다. 또한 network congestion 이 일어날 수 있고, 멀리 떨어진 클라이언트는 전송 속도가 느려질 수 있다. 또 같은 비디오의 복사본들이 링크를 통해 계속 보내질 수 있다. 가장 최대 단점은, 이 방식은 scale 이 불가능하다는 것이다. 

**option2 : 지리적으로 여러 장소에 비디오의 복사본들을 저장하고 서빙한다. (CDN)**
- enter deep : CDN 서버를 access network 로 보내는 것이다. 이것은 유저에게 가깝게 서버를 위치시킬 수 있다. 
- bring home : Inter Exchange Points(IXP) 를 access network 근처에 두는 것이다. 다른 인터넷 회사들이 연결되는 지점 (IXP) 근처에 content server 를 놓는 것이다. 이것은 관리하기는 쉽지만, 사용자 입장에서는 조금 delay 가 있을 수 있다. 

## Reference
- Computer Networking A Top Down Approach , 7th edition , ch.2-6
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 