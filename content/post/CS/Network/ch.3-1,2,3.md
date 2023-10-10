+++
author = "Soeun"
title = "[네트워크] Transport Layer -1 "
date = "2023-10-02"
description = "Transport Layer 의 배경과 UDP까지"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = ""
+++

이번에는 어플리케이션 계층과 네트워크 계층 사이에 있는 전송계층(transport layer) 에 대해 알아보겠다. 우선 전송계층과 어플리케이션 계층 간의 관계를 보고, UDP 까지 알아보겠다. 
다음 글에서 TCP 기술의 배경과 자세한 작동원리에 대해 알아보겠다. 

## 1 transport-layer service
전송계층은 어플리케이션 계층에서 서로 다른 호스트의 프로세스들이 소통할 수 있게 하는 logical communication 을 제공한다. 전송계층의 프로토콜은 end system 에서 작동한다. 전송계층은 메세지가 네트워크 계층에서 어떻게 전송되는지는 전혀 관여하지 않는다. 

- 보내는 쪽 : 어플리케이션의 메세지들을 segment들로 쪼개고, 네트워크 계층으로 보낸다. 
- 받는 쪽 : segment 들을 합쳐서 메세지로 복원하고, 어플리케이션 계층으로 전달한다. 

전송계층에서 사용하는 프로토콜에는 2가지가 있는데, TCP와 UDP 이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9afa1145-c02d-4eed-a967-d995025e4ec9)

### Transport vs. Network layer
- Network Layer : **호스트** 사이의 logical communication
- Transport Layer : **프로세스** 사이의 logical communication

예시를 하나 들어보자. 집에 있는 남동생이 미국에 사는 사촌에게 편지를 보낸다고 하자. 남동생에게 편지를 받아서 문 앞 우체통까지 넣어주는 일은 부모님이 한다. 그러면 우체부가 우편을 받아서 미국에 있는 사촌의 집앞에 전달한다. 그러면 사촌의 부모님이 우체통에서 편지를 꺼내서 편지를 사촌에게 전달한다. 

이것을 컴퓨터 네트워크의 세계에 대입하면, 아래와 같다. 
- 프로세스 = 남동생, 사촌
- 호스트 = 우리 집, 사촌 집 
- 우편 체계 = network layer
- 우리 부모님 / 사촌네 부모님 = transport layer

### Internet transport-layer protocols
어플리케이션 계층에게 2가지 transport-layer protocol을 제공한다고 했다. 이 2가지가 TCP, UDP 이다. 뒤에서 자세히 설명할 것이기에, 여기서는 간단히 알아보자.

- TCP : reliable, in-order delivery
	- congestion control 
	- flow control : 만약 받는 쪽에서 보내는 쪽에게 용량 때문에 못받겠다 하면 보내는 쪽의 속도를 늦춘다. 
	- connection setup
- UDP : unreliable, unordered delivery
