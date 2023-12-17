+++
author = "Soeun"
title = "[네트워크] www.google.com 을 입력하면 일어나는 모든 일"
date = "2023-11-21"
summary = "네트워크 연결부터, 웹 페이지가 보여지기까지"
categories = [
    "CS"
]
tags = [
    "network"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
math = true
slug = "what-happens-when-type-google"
series = ["Network"]
series_order = 16
+++
{{< katex >}}

이제 프로토콜에 대한 모든 것을 배웠으니, 큰 그림을 다시 보자. 한 학생(제임스)이 본인의 컴퓨터를 이더넷에 연결해서 google.com 웹페이지를 다운받는 다고 하자. 이 간단한 요청 안에 정말 많은 것이 숨겨져 있다.

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/345e856c-b37e-4b85-9d65-76bcf3d6654f">

## DHCP, UDP, IP, Ethernet

제임스가 노트북을 이더넷 케이블과 연결한다고 하자. 이 케이블은 학교의 이더넷 스위치에 연결되어 있다. 학교의 라우터는 ISP 에 연결되어 있고, 이 ISP 는 학교에 DNS 서비스를 제공한다. 

제임스가 처음으로 네트워크에 접속하면, 그는 여전히 아무것도 할 수 없다. 우선 IP 주소를 받아야 한다. 제임스의 노트북은 로컬 DHCP 서버로부터 DHCP 프로토콜을 사용해서 IP 주소를 얻는다. 

<img width="341" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c33fe7c8-b2aa-46ff-ab68-6d4af11024d8">

1. 제임스 노트북의 OS 는 DHCP request message 를 만들고, 이것을 UDP segment 에 넣고, 목적지 port 67 (DHCP server) 에게 보낸다. UDP segment 는 IP datagram 에 넣어지고, IP 목적지 주소는 브로드캐스팅 주소(255.255.255.255) 로 적는다. 소스IP 주소는 0.0.0.0 으로 적는다(아직 제임스가 IP 주소를 못 받았기 때문에) 
2. DHCP request 를 포함하는 IP datagram 은 이더넷 프레임 안에 캡슐화된다. 이더넷 프레임도 목적지 MAC 주소를 (FF-FF-FF-FF-FF-FF)로 적고, 브로드캐스팅한다. 프레임에 적인 소스의 MAC 주소는 제임스 노트북의 MAC 주소이다. 
3. 이더넷 프레임은 우선 제임스의 노트북에서 이더넷 스위치로 보내진다. 스위치는 들어오는 프레임을 모든 outgoing 인터페이스로 보낸다. 
4. DHCP 서버를 운영하고 있는 라우터에 프레임이 도착하고, 프레임 -> IP -> UDP 로 demux 된다. 
5. DHCP 는 (제임스 IP 주소, first-hop router 의 IP 주소, DNS 서버의 이름 & IP 주소) 를 포함하는 DHCP ACK 를 만든다. 
6. DHCP 서버에서 이것을 캡슐화해서 프레임을 LAN 에 전송하고 스위치는 self-learn 을 통해 이미 엔트리가 있으므로, 제임스의 노트북을 향한 인터페이스로만 frame 을 보낸다(unicast). 
7. 제임스의 노트북은 프레임을 받아서 demux 한다. 
8.  제임스의 DHCP client 는 본인의 IP 주소와 DNS 서버의 IP 주소를 기록해 놓는다. 그러면 제임스는 DHCP ACK 를 받아서 본인의 IP 주소를 알게 된다.

이 과정을 거치고 나면, 제임스는 IP 주소, DNS의 이름 & IP 주소, first-hop router 의 IP 주소를 알게 된다. 

## DNS and ARP

제임스가 www.google.com 을 웹 브라우저에 입력하면, 또 다시 긴 과정을 거치게 된다. 제임스의 웹 브라우저는 TCP socket 을 만들고, 이것은 HTTP request 를 보낼 때 사용된다. 소켓을 만들기 위해서는 제임스의 노트북은 www.google.com 의 IP 주소를 알아야 한다. 이때 DNS 프로토콜을 사용해서 name-to-IP 번역 서비스를 사용한다. 

1. 제임스 노트북의 OS 는 **DNS query message** 를 만든다. question section 에 "www.google.com" 이 들어간다. 이 DNS message 는 UCP segment 에 넣어지고, 목적지 port 53 (DNS server) 로 보내진다. UDP segment 는 IP datagram에 넣어지고, IP 목적지 주소는 DNS 서버의 IP 주소이다. 
2. 제임스의 노트북인 이 datagram 을 이더넷 프레임으로 캡슐화한다. 이 프레임은 학교의 gateway router 로 보내진다. 여기서 제임스는 gateway router 의 IP 주소는 알지만, MAC 주소는 아직 알지 못한다. gateway router 의 MAC 주소를 알아내려면 ARP 를 사용해야 한다. 
3. 제임스의 노트북은 **ARP query message** 를 만든다. 목적지 IP 주소는 gateway router 의 IP 주소이고, MAC 주소는 브로드캐스팅 주소인 FF-FF-FF-FF-FF-FF 이다. 이더넷 프레임을 스위치에 보내고, 스위치는 이 프레임을 연결된 모든 디바이스에 보낸다. 
4. gateway router 는 ARP query message 를 받아서 여기에 있는 목적지 IP 주소가 본인의 IP 주소와 일치함을 발견한다. gateway router 는 **ARP reply** 를 준비하고, 여기에는 본인의 MAC 주소를 포함한다. 이것을 이더넷 프레임안에 넣는데 이 프레임의 목적지 IP 주소는 제임스의 IP 주소이다. 스위치로 보내면 이 프레임을 제임스의 노트북으로 전송한다. 
5. 제임스의 노트북은 gateway router의 MAC 주소를 포함하는 ARP reply 를 받는다. 
6. 최종적으로 제임스의 노트북은 gateway router의 MAC 주소를 포함하는 DNS query 를 보낼 수 있다. IP datagram은 이때 IP 목적지로 DNS 서버의 IP 주소를 적고, 이더넷 프레임은 목적지로 gateway router 의 IP 주소를 적는다. 

## Intra-Domain Routing to the DNS server

1. gateway router 는 프레임을 받아서 DNS query 를 포함하는 IP datagram 을 추출한다. 라우터는 IP datagram 에 적힌 목적지 IP 주소(DNS 서버의 IP 주소) 가 있는지 forwarding table 에서 찾고, 어느 링크로 보낼지 결정한다. IP datagram 은 이더넷 프레임에 넣어지고, 링크로 전달된다. 
2. 학교 네트워크의 가장 끝에 있는 라우터가 프레임을 받고, IP datagram 을 꺼내고, 이 IP 주소(DNS 서버의 IP 주소) 를 검사하고 outgoing interface 를 결정한다. 
3. 최종적으로 DNS 서버에 DNS query 를 포함하는 IP datagram 이 도착한다. DNS 서버는 DNS query message 룰 추출한 후, "www.google.com" 이 있는지 DNS database 를 찾아본다. 그 후  "www.google.com" 의 IP 주소를 포함하는 DNS resource record 가 있다는 것을 발견한다. 이떄 이 레코드는 캐싱되어 있었다고 가정한다. DNS 서버는 (hostname-IP주소) 가 적혀있는 **DNS reply message** 를 작성하고, DNS reply 를 UDP segment 에 넣은 다음 제임스의 IP 주소가 목적지 적힌 IP datagram 에 넣는다. 이 datagram 은 라우터를 통과해서 제임스의 노트북에 도착할 것이다. 
4. 제임스의 노트북은 DNS message 에서 "www.google.com" 의 IP 주소를 추출한다. 이제 제임스의 노트북은 "www.google.com" 서버에 연결할 준비가 되었다 !!

## Web Client Server Interaction  : TCP and HTTP 

1. 제임스의 노트북이 이제 "www.google.com"  의 IP 주소를 알기 때문에, **TCP socket** 을 만들 수 있다. 이것은 **HTTP GET** 메세지를 만드는데 사용될 것이다. 제임스가 TCP socket 을 만들때, 제임스 노트북에 있는 TCP 는 우선 "www.google.com"  의 TCP와 **3-way handshake** 를 해야 한다. 따라서 제임스의 노트북은 우선 **TCP SYN segment** 를 만들고, 목적지 port 는 80 (for HTTP) 이다. 여기 안에 목적지 IP 주소로 구글의 IP 주소가 적힌 IP datagram을 넣는다. 이 datagram 은 gateway router 의 MAC 주소가 목적지 MAC 주소로 적힌 프레임안에 캡슐화되고 스위치로 보내진다. 
2. 교내 네트워크 안에 있는 라우터들은 fowarding table 을 이용해서 "www.google.com" 에게 datagram 을 포워딩한다. 이때 다른 AS 로 보낼 때는 inter-domain link protocol 인 BCP 를 이용한다. 
3. 결국에는 "www.google.com"  에게 TCP SYN 을 포함하는 datagram 이 도착한다. TCP SYM 은 port 80 의 소켓으로 demux된다. 이제 제임스의 노트북과 구글의 HTTP server 사이에 TCP 연결이 완료되었다. 이때 **TCP SYNACK segment** 가 만들어지고, 제임스 노트북의 IP 주소를 목적지로 한datagram 에 넣어진다. 구글과 first-hop 라우터를 연결하는 링크로 보내진다.
4.  TCP SYNACK 를 포함하는 datagram 은 구글 -> 교내 네트워크 -> 제임스 노트북의 이더넷 카드로 보내진다. 이 datagram 은 아까 만들어진 TCP socket 으로 demux 되고, 이제 연결상태에 진입한다. 
5. 이제 제임스 노트북에 있는 socket 이 bytes 들을 "www.google.com" 에게 보낼 상태가 되자, 제임스의 노트북은 **HTTP GET message** 를 만든다. 이 HTTP 메세지는 소켓으로 들어가고, TCP segment 의 payload 가 된다. TCP segment 은 datagram 에 넣어지고, "www.google.com" 로 전달된다. 
6. "www.google.com" 의 HTTP server 가 HTTP GET message 를  TCP socket 에서 읽고, HTTP response message 를 만든다. 여기에는 요청한 웹페이지 컨텐츠가 body 로 들어간다. 이 메세지를 TCP socket 으로 보낸다. 
7. HTTP reply message 를 포함하는 datagram 은 구글-> 교내 네트워크 -> 제임스 노트북으로 전달된다. 제임스의 웹 브라우저는 소켓으로부터 HTTP response message 를 읽고, html 을 추출해서 웹 페이지를 보여준다 ! 


