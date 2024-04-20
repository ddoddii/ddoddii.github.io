+++
author = "Soeun"
title = "[네트워크] HTTP 1.0 vs 1.1 vs 2.0 vs 3.0"
date = "2024-02-20"
summary = "HTTP의 발전 과정"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
slug = "http-versions"
+++

HTTP(Hypertext Transport Protocol) 는 웹 상에서 정보를 주고 받기 위한  어플리케이션 계층 프로토콜입니다. HTTP는 클라이언트가 요청을 하기 위해 연결을 연 다음 응답을 받을때 까지 대기하는 전통적인 [클라이언트-서버 모델](https://en.wikipedia.org/wiki/Client%E2%80%93server_model)을 따릅니다. HTTP는 [무상태 프로토콜](https://en.wikipedia.org/wiki/Stateless_protocol)이며, 서버가 두 요청 간에 어떠한 데이터(상태)도 유지하지 않습니다.

처음에는, HTTP는 인터넷 상에서 서버와 클라이언트가 소통을 하기 위한 용도로 만들어졌습니다. 가장 첫 HTTP 버전인 HTTP/0.9 는 오직 GET 메서드만 있었으며, ASCII 데이터만 전송 가능했습니다. 본 포스팅에서는 HTTP가 0.9, 1.0, 1.1, 2.0, 3.0 까지 발전하며 달라진 점들을 살펴보겠습니다. 

## HTTP/0.9

> The one-line protocol

HTTP 의 가장 초기 버전은 굉장히 간단했습니다. 딱 한줄로, **GET 메서드**와 **리소스 경로**만 있었습니다.

```HTTP
GET /index.html
```

응답도 굉장히 간단했습니다. 파일 자체로만 이루어져 있었습니다.

```HTML
<html>
  A very simple HTML page
</html>
```

HTTP headers 가 없었고, 오직 HTML 파일만 전송가능했습니다. 

## HTTP/1.0

> Building extensibility

HTTP/0.9 에서는 GET 메서드만 지원했었습니다. 그러나 인터넷이 발전함에 따라 정보를 가져오는 다른 방법들도 필요해졌습니다. HTTP/1.0 에서 도입된 점들은 아래입니다. 
- **Header** : HTTP/0.9 에서는 메서드와 리소스 이름만 있었습니다. HTTP/1.0 에서는 [HTTP header](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers) 를 도입해서, 더 다양한 메타데이터를 전송할 수 있게 했습니다. 
- **Versioning** : HTTP 의 어느 버전인지 명시할 수 있게 했습니다.
- **Status code** : 응답에 대한 상태 코드를 만들었습니다.
- **Content-type** : 이 필드 덕분에 HTML 파일 말고도 다른 타입의 다큐먼트도 전송 가능해졌습니다.
- **New methods** : GET 말고도 POST, HEAD 메서드가 추가되었습니다. 

```HTTP
GET /mypage.html HTTP/1.0
User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)

200 OK
Date: Tue, 15 Nov 1994 08:12:31 GMT
Server: CERN/3.0 libwww/2.17
Content-Type: text/html
<HTML>
A page with an image
  <IMG SRC="/myimage.gif">
</HTML>
```

따라서 HTTP/1.0 은 초기 버전보다 훨씬 더 확장가능해졌습니다. HTTP/1.0 에서 가장 중요한 점은 HTTP headers 의 도입과, 새롭게 추가된 HTTP methods 입니다. HTTP header 는 클라이언트가 다양한 파일 타입을 주고 받고, 메타데이터를 주고받을 수 있게 했습니다. 새로운 메서드들은 클라이언트가 서버에게 정보를 보낼 수 도 있고([POST](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)), 메타데이터를 받을 수 도 있게 되었습니다([HEAD](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)). 

## HTTP/1.1

> The standardized protocol 

HTTP/1.1 은 HTTP/1.0 이 출시된 몇달 후(1997년)에 출시되었습니다. HTTP/1.1 이 업그레이드 한 부분들은 다음과 같습니다.
- **Host header** : 호스트 헤더로 인해, 하나의 물리 서버(하나의 IP주소)에서 여러 가상 호스트를 구분할 수 있게 되었습니다. 
- **Persistent connections** : HTTP/1.0 에서 각 요청/응답 페어마다 새로운 연결을 수립해야 했습니다. HTTP/1.1 에서는 하나의 연결로 여러 개의 요청/응답을 처리할 수 있게 되었습니다.
- **Pipelining** : 파이프라이닝을 통해 첫번째 요청에 대한 응답을 받기 전에 두번째 요청을 보낼 수 있게 되었습니다. 
- **Continue status** : 서버가 응답을 할 수 있는 상태인지 확인하기 위해, 첫번째 요청에서는 요청 헤더만 보내 continue 상태 코드(100)을 받는지 확인했습니다. 
- **New methods** : 6개의 새로운 메서드(PUT, PATCH, DELETE, CONNECT, TRACE, OPTIONS) 가 추가되었습니다. 

하나의 연결로 보낸 요청들의 예시는 아래와 같이 생겼습니다.

```HTTP
GET /en-US/docs/Glossary/Simple_header HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/en-US/docs/Glossary/Simple_header

200 OK
Connection: Keep-Alive
Content-Encoding: gzip
Content-Type: text/html; charset=utf-8
Date: Wed, 20 Jul 2016 10:55:30 GMT
Etag: "547fa7e369ef56031dd3bff2ace9fc0832eb251a"
Keep-Alive: timeout=5, max=1000
Last-Modified: Tue, 19 Jul 2016 00:59:33 GMT
Server: Apache
Transfer-Encoding: chunked
Vary: Cookie, Accept-Encoding

(content)

GET /static/img/header-background.png HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/en-US/docs/Glossary/Simple_header

200 OK
Age: 9578461
Cache-Control: public, max-age=315360000
Connection: keep-alive
Content-Length: 3077
Content-Type: image/png
Date: Thu, 31 Mar 2016 13:34:46 GMT
Last-Modified: Wed, 21 Oct 2015 18:27:50 GMT
Server: Apache

(image content of 3077 bytes)
```

HTTP/1.1 은 꽤 오랜 기간동안 표준 프로토콜로 사용되었습니다. 그만큼 확장성과 안정성이 뛰어났습니다. 이로 인해 새로운 헤더와 메서드들을 만들기 쉬웠습니다. 

### HTTP for secure transmissions

보안을 위해 HTTP 를 기본적인 TCP/IP 위에서 전송하는 것보다, 보안을 위한 레이어인 SSL 가 고안되었습니다. 현재는 SSL의 발전된 형태인 TLS가 사용되고 있습니다.

### Using HTTP for complex applications

특수한 어플리케이션들을 위해 HTTP 프로토콜의 발전된 형태도 고안되었습니다. 
- [Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) : 서버가 브라우저에게 메세지를 푸쉬할 수 있습니다.
- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) : 기존의 HTTP 연결을 업그레이드 하여 실시간 양방향 소통을 할 수 있습니다. 

## HTTP/2 

> A protocol for greater performance

시간이 지나면서, 웹 페이지는 점점 더 복잡해졌습니다. 웹 상에서는 단순 HTML 페이지 뿐만 아니라 여러 가지 미디어도 포함되었습니다. 따라서 HTTP 요청-응답에 상당한 오버헤드가 필요해졌습니다. 따라서 2010년대 초반에 구글이 SPDY 라는 실험적인 프로토콜을 구현했습니다. SPDY 는 응답 속도를 개선했고, 중복된 데이터 전송을 개선했습니다. 

HTTP/2 가 HTTP/1.1 에 비해 달라진 점은 아래와 같습니다 :
- **Request multiplexing** : HTTP/1.1 은 순차적인 프로토콜입니다. 이 말은, 요청한 순서대로 응답이 도착해야 한다는 의미입니다. 이것은 HOL 문제를 야기했고, 따라서 HTTP/2 에서는 멀티플렉싱을 도입하여 응답이 요청한 순서대로 오지 않아도 되게끔 했습니다. 
- **Request prioritization** : 요청에 우선순위를 지정해서, 시간이 짧게 걸리는 CSS 파일을 먼저 받고, 오래 걸리는 JS 파일들을 나중에 받을 수 있도록 하여 응답속도를 개선했습니다.
- **Automatic compressing** : HTTP/2.0 에서는 GZip 을 이용한 압축을 자동적으로 수행합니다. 
- **Connection reset** : 서버와 클라이언트 간 연결이 끊기면, 바로 새로운 연결을 시작하게끔 했습니다. 
- **Server push** : 서버가 너무 많은 요청을 받는 것을 방지하기 위해, 서버 푸쉬가 생겼습니다. 서버는 클라이언트가 요청한 리소스 뿐만 아니라 앞으로 필요할 것 같은 리소스를 예측해서 하나의 요청에 함께 응답을 보냅니다. 
- **Binary protocol** : HTTP/1.1 까지는 텍스트 형태의 프로토콜이었지만, HTTP/2 부터는 바이너리 형태입니다. 

## HTTP/3

> HTTP over QUIC

HTTP/2 로 인해 성능 향상이 되었지만, 여전히 HTTP 는 TCP 기반이므로 TCP 수준에서 HOL 문제는 해결되지 않았고, RTT 가 너무 오래 걸렸습니다. 따라서 HTTP/3 에서는 TCP 대신 UDP 기반인 **QUIC** 위에서 작동하는 프로토콜을 설계했습니다. 

QUIC(Quick UDP Internet Connections)는 전송 계층 프로토콜로, 멀티플렉싱과 자체 보안 기능을 내장하고 있습니다. 또한 TCP 보다 훨씬 빠른 핸드셰이크 프로세스를 가지고 있어, 연결을 훨씬 빠르게 맺을 수 있습니다. 그리고 UDP 상에서 여러 개의 스트림을 만들어서, 각 스트림마다 패킷 loss 를 독립적으로 처리합니다. 따라서 패킷을 잃어버려도, 전체가 영향을 받는 것이 아니라 그 스트림만 영향을 받습니다. 



## Reference
- https://developer.mozilla.org/ko/docs/Web/HTTP
- https://www.baeldung.com/cs/http-versions
- https://developer.mozilla.org/ko/docs/Web/HTTP/Headers
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP
- https://www.baeldung.com/cs/rest-vs-http