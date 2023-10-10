+++
author = "Soeun"
title = "[네트워크] Domain Name System(DNS)"
date = "2023-09-30"
description = "인터넷의 Directory System 인 DNS"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
+++

## DNS

사람은 이름으로 구분할 수 있다. 그러면 host 들을 어떻게 구분할까 ? 사람들은 www.google.com , www.naver.com 등 호스트의 이름을 보고 구분할 수 있다. 하지만, 라우터들은 단순히 글자의 이름을 보고 구분하기 어려울 것이다. 그래서 호스트는 고유한 IP 주소로 구분한다. 

### DNS가 제공하는 서비스

호스트를 구별하는 2가지가 있다고 했다 - hostname, IP 주소. 라우터는 IP 주소를 선호하고, 사람은 의미를 가지고 있는 hostname 을 좋아한다. 그러면 이 둘 사이를 번역해주는 것이 필요하다 ! 그래서 나온 것이 DNS 이다. 

DNS는 - (1) DNS 서버의 계층구조 하에 있는 분산된 데이터베이스 , (2) 호스트와 네임 서버가 이름을 번역하기 위해 소통하는 어플리케이션 계층 프로토콜이다. 

DNS는 흔히 다른 어플리케이션 계층 프로토콜(HTTP, SMTP 등)에 의해 사용된다. 사용자가 브라우저에 URL(www.naver.com) 을 입력하면, 사용자의 호스트가 웹 서버에 request 를 보내기 위해서는 웹 서버의 IP 주소를 알아야 한다. 그 때 이 URL -> IP 주소를 번역하기 위해 DNS 를 사용한다. 

이 번역 기능 외에도 DNS는 다른 기능들을 제공한다. 
- Host aliasing
	
	실제로 호스트 이름은 매우 복잡하고 다양할 수 있다. 서울에 있는 사용자가 www.google.com 에 접속하면 실제 호스트 이름은 www.google-kr-seoul.com (실제는 아님..) 이 될 수 있다. 이러한 실제 호스트 이름을 canonical hostname 이라고 한다. 이러한 주소들을 사용자가 보는 호스트 이름으로 바꿔준다.

- Mail Server aliasing
	
	메일 주소에 대해서도 위와 비슷한 기능을 제공한다.

- Load distribution
	
	접속량이 많은 사이트를 생각해보자. (cnn.com) 이러한 사이트는 여러 대의 서버가 호스팅을 한다. 그러면 하나의 canonical hostname 에 여러 개의 IP 주소가 있을 수 있다. DNS 데이터베이스는 IP 주소들의 세트를 저장한다. 그래서 그 사이트에 대한 요청이 왔을 때, DNS rotation 을 통해 로드를 분산한다. 

### DNS는 어떻게 작동하는가? 

여기서는 hostname - IP address 번역 서비스에 집중한다. 

단순 번역 서비스라면, 하나의 DNS 서버에 모든 매핑 정보를 저장하고, 요청이 올 때마다 찾아서 반환하면 되는 것이 아닌가 ? 라고 생각할 수 있다. 하지만 역시 분산화 하는 데에는 이유가 있다. 하나의 서버로 통일하면, 더 이상 scale 하기 불가능하다. 그리고 사용자의 거리가 멀면 응답 받는 속도가 느려진다. 

그래서 DNS는 **Distributed, Hierarchical Database** 이다. 

#### DNS 서버의 3가지 클래스

DNS 서버에는 3가지 클래스가 있다 - **root, top-level domain (TLD), authoritative**. 아래와 같은 구조로 계층화 되어 있다. 

<img width="449" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1d5a01f9-3335-4207-85f9-26e84e992fc2">

예를 들어, www.amazon.com 이라는 호스트이름을 번역한다고 하자. 그러면 root DNS server 가 com 을 위한 TLD DNS server 의 IP 주소를 반환한다. 이 TLD DNS server 는 또 amazon.com 의 authoritative server 의 IP 주소를 반환하고, 마지막으로 authoritative server 는 실제 www.amazon.com 의 IP 주소를 반환한다. 

#### Local DNS name server

이 로컬 DNS 서버는 위의 계층에 딱 들어가지는 않는다. 그러나 각 ISP 는 하나씩 가지고 있다. 호스트가 DNS 쿼리를 날리면, 쿼리는 제일 처음 로컬 DNS 서버로 보내진다. 이 로컬 DNS 에는 최근의 번역 정보가 캐시로 저장되어 있다. 이것을 **DNS caching** 이라고 한다. 

DNS 가 쿼리하는  2가지 방식

- iterated query

<img width="349" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f7373ea6-bc8e-41ba-95bc-32ca3d4970d2">

- recursive query

<img width="349" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/86b4998a-3e6a-4034-a2ac-b2c6865eaa30">

### DNS Records and Messages

#### DNS RR

DNS 는 **'Distributed database storing resource records(RR)'** 이다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bb351ffe-adf3-4f4d-bef5-c8e830c4c20b)

name, value 의 의미는 Type 에 따라 달라진다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cce37a0c-1c26-4b28-931a-df7822bf7f10)

#### DNS Messages

DNS 는 같은 format 으로 메세지를 query 하고 reply 한다. 

<img width="488" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2a597af5-ebe6-481c-8335-cf7afccc517e">


--------------------------------------------------------
## DNS 에 대한 질문들
 
- **DNS는 몇 계층 프로토콜인가요?**

    DNS는 어플리케이션 계층 프로토콜이다. DNS 는 메세지를 전달하기 위해 하위 transport layer 프로토콜을 사용하므로, 어플리케이션 계층 프로토콜이다.

- **UDP와 TCP 중 어떤 것을 사용하나요?**

    DNS는 몇 가지 케이스를 제외하고는 일반적으로 UDP 를 사용한다. 
    
    가장 중요한 이유는,  UDP 는 TCP 와 달리 연결 설정을 할 필요가 없이 바로 데이터를 전송하므로, 속도가 빠르다. 또 UDP 는 512 바이트를 넘어가지 않는 패킷만 전송이 가능한데, DNS가 다루는 데이터의 사이즈가 작으므로 적합하다. 그래서 속도가 빠른 UDP 를 우선으로 선택한다. 

    그렇지만 UDP 는 연결을 보장하지 않고 데이터를 전송하므로 데이터가 제대로 갔는지 확인하지 않는다. 그래서 데이터가 손실 될 수 있다. 그런데도 UDP 를 쓰는 이유는, UDP 는 빠른 연결과 상태를 저장하지 않는 stateless 특성 덕분에 더 많은 클라이언트들에게 데이터를 전송해줄 수 있다. UDP 의 안전이 의심되면, 어플리케이션 계층에 보호 수단을 더할 수 있다. 

    DNS 가 UDP 를 사용해서 받을 수 있는 공격으로는 DNS Spoofing 공격이 있다. 보낸 쪽이 데이터가 잘 갔는지 확인하지 않으므로 중간에서 가로채서, 다른 URL 을 보내주는 것이다. 이 URL 에 악성 코드가 심어져 있으면 유저는 당할 수 있다. 

    DNS가 TCP 를 사용하는 경우는 아래의 케이스이다. 
    - Zone Transfer 
    - Large DNS Responses : UDP 는 512 byte 이하의 데이터만 전송한다. 그래서 큰 response 는 TCP 를 사용한다. 
    - DNS over TCP (DoT) and DNS over TLS(DoT) : 보안을 위해, TCP 기반 프로토콜을 이용해 암호화할 수 있다. 
    - Firewall and Network Restrictions : 방화벽이나 네트워크가 UDP 를 제한하면, TCP 를 사용한다. 

- **DNS Recursive Query, Iterative Query가 무엇인가요?**

    Recursive Query : 로컬 DNS 서버 -> root DNS 서버 -> TLD DNS 서버 -> authoritative DNS 서버 에게 차례대로 쿼리를 하는 것이다. 

    Iterative Query : 로컬 DNS 서버가 계속해서 순차적으로 root DNS 서버,  TLD DNS 서버, authoritative DNS 서버 에게 물어보는 것이다. 

- **DNS 쿼리 과정에서 손실이 발생한다면, 어떻게 처리하나요?**

    DNS가 UDP를 사용해서 쿼리에 손실이 발생하면 어떻게 하냐는 질문이 많았다. 하지만 답변들을 찾아보니, 놀랍게도 '어쩔 수 없다'는 것이다. 잃어버린 데이터는 그냥 잃어버린 것이다. 그저 DNS 쿼리에 손실이 발생해서 생길 문제를 최소화하는 것이 최선이다. DNS 서버를 여러 대를 쓰거나, 아예 호스트네임 말고 IP 주소를 쓰는 것.. 이다. 


- **DNS 레코드 타입 중 A, CNAME, AAAA의 차이에 대해서 설명해주세요.**

    Resource Records(RR) 는 hostname - IP address 를 매핑해주는 데이터셋이다. DNS reply 메세지는 하나 이상의 RR을 담고 있다. RR는 4개 영역을 담고 있는 튜플이다.
    
     (Name, Value, Type, TTL)

     여기서 Name, Value 의 의미는 Type 에 따라 달라진다. 
    - A : name- hostname, value - IPV4 address 
      - (www.naver.com , 223.130.195.200)
    - CNAME : name - alias name, value - canonical name 
      - (www.ibm.com, servereast.backup2.ibm.com)
    - AAAA : name - hostname, value - IPV6 address

- **hosts 파일은 어떤 역할을 하나요? DNS와 비교하였을 때 어떤 것이 우선순위가 더 높나요?**

    호스트 파일은 각각의 도메인네임과 이들의 IP주소를 하나하나 대응시켜서 저장해 놓은 텍스트파일이다. 인터넷이 세상에 나온지 얼마 안된 초기에는 호스트파일이 유용했겠지만, 시간이 지날수록 많은 서버들이 생겨났을 것이고, 새롭게 등장한 서버의 정보(이름과 IP주소의 나열이겠지만)를 계속 업데이트 하기에는 좀 벅찼을 것이다. 또한 서버의 IP주소가 바뀌었다면 호스트파일도 수정해야 하는 번거로움이 있다. 이것이 네임서버가 생긴 이유중의 하나일 것이다.

    우리가 브라우저에 검색을 하면, 우선 호스트 파일을 먼저 검색한다. 그 후 호스트 파일에 해당 IP 주소가 없으면 DNS서버에 쿼리를 날린다. 따라서 DNS서버 쿼리보다는 호스트파일의 우선순위가 높다. 

    실제로 mac 에서 호스트파일을 확인하려면, 터미널에 `sudo vim /private/etc/hosts` 을 입력하면 된다. 여기에 hostname- 실제 IP 주소를 매핑한 것을 저장해주면 된다. 


## Reference
- https://www.cloudns.net/blog/dns-use-udp/
- https://www.techtarget.com/searchnetworking/answer/Since-DNS-uses-UDP-instead-of-TCP-if-a-packet-is-lost-there-is-no-automatic-recovery
- https://serverfault.com/questions/404840/when-do-dns-queries-use-tcp-instead-of-udp
- https://blog.stories.pe.kr/530
- Computer Networking A Top Down Approach , 7th edition
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 
