+++
author = "Soeun"
title = "[네트워크] Simple Mail Transfer Protocol(SMTP)"
date = "2023-09-29"
summary = "메일을 보내기 위한 프로토콜"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/408cc122-3a08-4657-a3b5-ebd316196b1a"
slug = "SMTP"
series = ["Network"]
series_order = 4
+++
{{< katex >}}

## 메일을 주고 받는 프로토콜

메일을 주고 받을 때, 크게 3가지 구성요소로 나눌 수 있다. 
- user agents
	- 메일 메시지를 작성, 읽는 사람
	- e.g. Gmail client
- mail servers
	- mailbox : 유저를 위해 메일을 보관
	- message queue : 보낼 메일을 저장하는 큐
- SMTP (simple mail transfer protocol)
	- 메일 서버 사이에서 메일을 전송하고 받기 위한 프로토콜

### SMTP

SMTP 는 메일을 주고 받기 위해 TCP 를 사용한다. 왜냐면 메일의 내용이 손상이 되면 안되기 때문이다. 메일을 보낼 때 시나리오를 생각해보자.

1. 내가 이도현에게 이메일을 보내고자 한다. 이도현님의 이메일 주소를 찾아서, 메세지를 적고, user agent(gmail 등) 을 이용해서 메세지를 보내라고 한다.
2. 내 user agent 는 내 mail server 에게 이 메세지를 보낸다. 이 메세지는 message queue 에 저장된다.
3. 내 SMTP 는 message queue 에 저장되어 있는 메세지를 발견하고, 이도현 님의 mail server 와 TCP 연결을 한다. 
4. SMTP handshaking 후에, 내 SMTP client 는 TCP 연결을 통해 메세지를 보낸다. 
5. 이도현님의 메일 서버는 메세지를 받고, 이 메세지를 mailbox 에 보관한다. 

<img width="669" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a0dcf86e-9cbd-4bdc-a156-fffb7f4bfc6e">

SMTP 는 이메일을 보내기 위해 TCP 를 사용하고, port 25번을 사용한다. 그리고 HTTP 처럼 command/response 인터랙션이 이루어진다. 
- commands : ASCII text 형태
- response : status code and phrase

### SMTP vs. HTTP

HTTP 는 pull protocol 이다. 사용자는 web 에게 정보를 요청하는 것으로 Request 를 보낸다. 이때 TCP 연결은 파일을 받고 싶은 쪽에서 시작한다. 반면에 SMTP 는 push protocol 이다. 이때 TCP 연결은 파일을 보내고 싶은 쪽에서 시작한다.

HTTP 는 각 object(text , image ...) 를 response message 에 캡슐화한다. SMTP 는 메세지의 object 들을 하나의 메세지에 넣는다. 

### Mail access Protocols

메일을 받는 쪽에서, 메일 서버에서 user agent 까지 메일을 읽을 때는 mail access protocol 을 사용한다. 여기에는 POP, IMAP, HTTP 가 있다. 요즘은 web 기반 메일(gmail, naver mail..) 을 써서 HTTP 를 가장 많이 사용한다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/009fb50e-6ad5-4e98-b6dc-c6c403bae32f)

## Reference
- Computer Networking A Top Down Approach , 7th edition
- 2023-2 컴퓨터 네트워크 , 이수경 교수님 강의안 