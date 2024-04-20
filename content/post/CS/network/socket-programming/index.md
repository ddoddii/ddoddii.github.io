+++
author = "Soeun"
title = "[네트워크] 소켓은 어떻게 작동할까?"
date = "2024-04-01"
summary = "C언어로 본 소켓 프로그래밍"
categories = [
    "CS"
]
tags = [
    "네트워크"
]
slug = "socket-programming"
series = ["Network"]
series_order = 18
+++

네트워크 공부를 하다 보면, 소켓에 대해 많이 접하게 됩니다. 본 포스팅에서는 소켓이 무엇이며, 어떤 역할을 하는지 알아보겠습니다. 또한 C언어로 실제 클라이언트-서버 사이 소통을 구현해보겠습니다.

## 데이터를 송신,수신하는 과정은?

클라이언트와 서버 간에 데이터를 송신,수신 할때 정확히 어떤 일이 일어날까요? 메세지를 송신,수신하는 역할은 정확히 우리 컴퓨터의 운영체제가 담당합니다. 즉, 사용자가 브라우저를 통해 웹 서버에 어떤 정보를 요청해도, 실질적으로 메세지를 보내는 역할은 운영체제가 합니다. 정확히 말하자면, 브라우저는 해당 웹 서버에 메세지를 송신하도록 Socket 라이브러를 호출하여, OS 내부에 있는 **프로토콜 스택**에 메세지 송신을 의뢰합니다.

Socket 라이브러리를 이용한 데이터 송수신을 비유하자면, 마치 송신/수신하는 컴퓨터 사이에 파이프라인과 같은 데이터의 통로가 있다고 생각하면 됩니다. 서버와 클라이언트에 각각 소켓이 있고, 소켓 사이가 파이프와 같은 것으로 연결되고, 데이터가 파이프 사이로 흐릅니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/058782a0-8a5b-4e68-a514-e643787983fa)

파이프는 실제로 물리적으로 파이프가 만들어지는 것이 아니라, 논리적인 커넥션 개념입니다. 소켓은 파이프 양끝에 있는 데이터의 출입구로, 네트워크를 통해 다른 시스템과 통신할 수 있게 해주는 인터페이스의 역할을 합니다. 논리적인 연결에서 각 소켓은 특정 IP주소와 포트 번호로 바인딩되어서, 네트워크 상 다른 소켓과 구별됩니다. 비유하자면, 연결을 맺는다는 것은 "나 너에게 이 IP주소를 가지고 xx번 포트 번호로 보낼거니까 거기로 데이터 들어가면 난 줄 알아!" 라고 약속을 하는 개념입니다. 

하지만 이전에 파이프를 만드는 작업부터 해야 합니다. 이때 실제로는 서버 측에서 소켓을 만들고, 소켓에 클라이언트가 파이프를 연결하기를 기다립니다. 그러면 클라이언트 측도 소켓을 만들고, 서버측의 소켓에 연결합니다. 

즉 4가지 단계로 요약할 수 있습니다. 
1. 소켓 만들기 (소켓 작성 단계)
2. 서버측의 소켓에 파이프 연결하기 (접속 단계)
3. 데이터를 송신,수신하기 (송,수신 단계)
4. 파이프를 분리하고 소켓 말소하기(연결 끊기 단계)

## 어플리케이션이 OS에게 데이터 송신,수신 의뢰하기 

어플리케이션은 데이터를 송신,수신하기 위해 OS에게 의뢰해야 합니다. (실질적인 메세지 송수신 과정은 OS만 할 수 있습니다!) 이때, 어플리케이션은 Socket 라이브러리 프로그램을 호출합니다. 이때 여러 과정을 거치는데, 자세히 살펴보겠습니다.

1. **소켓 생성하기**

클라이언트 측에서 소켓을 만드는 것은, socket 라이브러리의 `socket` 을 호출합니다. 소켓이 생기면 **디스크립터**가 만들어지는데, 이것은 소켓을 식별하기 위해 사용됩니다. 

2. **파이프를 연결하는 접속 단계**

다음으로는 소켓을 서버측의 소켓에 접속하도록 OS의 프로토콜 스택에 의뢰합니다. 이때 socket 라이브러리의 `connect` 를 사용합니다. 이때 **디스크립터, 서버의 IP주소, 포트번호** 3가지 값이 필요합니다. 포트번호는 서버에 연결했을때, 서버의 어떤 소켓을 사용할지를 결정합니다. 그러면 서버는 클라이언트측의 소켓 번호를 어떻게 알 수 있을까요? 이것은, 먼저 클라이언트측의 소켓의 포트 번호는 소켓을 만들 때 프로토콜 스택이 적당한 값을 골라서 할당합니다. 그리고 이 값을 프로토콜 스택이 서버측에 접속할 때 통지합니다. 


3. **메세지를 송수신하는 단계**

소켓이 연결되면, 소켓에 데이터를 넣으면 상대측에 데이터가 도착합니다. 이때 `write` 를 사용합니다. 어플리케이션은 송신할 데이터를 메모리에 준비하고, `write` 를 호출할 때 디스크립터와 송신 데이터를 저장합니다. 그러면 프로토콜 스택이 송신 데이터를 서버에 송신합니다. 그러면 서버는 그 데이터를 받고, 적절한 응답 메세지를 보냅니다. 수신할 때는 `read` 를 사용합니다. 수신한 응답 메세지를 저장하기 위해 메모리 영역을 지정하는데, 이 영역을 **수신 버퍼**라고 합니다. 응답 메세지를 `read` 로 받아서 수신 버퍼에 저장하고, 이 메세지를 다시 어플리케이션에게 전달해줍니다. 

4. **연결 끊기 단계**

Socket 라이브러리의 `close` 를 사용하여 연결을 종료합니다. 그러면 소켓도 말소되고, 소켓 사이의 연결도 끊깁니다. 

## C로 보는 소켓 프로그래밍

이제 C를 사용해서, 실제 소켓 프로그래밍이 어떻게 이루어지는지 봅시다. 

서버측의 소켓은 특정 IP의 특정 포트 번호를 listen 하고, 클라이언트 측의 소켓은 연결을 시도합니다. 서버는 listener socket 를 만듭니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/91cd46df-a19c-4f19-a983-2358b3b80f18)

### Stages for Server

**서버** 쪽에서 준비해야 하는 단계들을 봅시다.

1. **Socket Creation**

```c
int sockfd = socket(domain, type, protocol)
```

- sockfd : 소켓 디스크립터 
- domain : 커뮤니케이션 도메인
- type : 연결 타입. SOCK_STREAM 은 TCP를 뜻하고, SOCK_DGRAM 은 UDP연결을 뜻합니다.
- protocol : IP를 위한 프로토콜 값 


2. **Setsockopt**

```c
int setsockopt(int sockfd, int level, int optname,  const void *optval, socklen_t optlen);
```

이 단계는 옵셔널입니다. 이 단계에서는 디스크립터에 의해 지정된 소켓을 조작하기 위한 몇가지 옵션을 제공합니다. 이 옵션으로 주소와 포트 번호를 재사용할 수 있습니다. 

3. **Bind**

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

소켓이 만들어진 후, bind 함수는 소켓을 addr 에 명시된 주소와 포트 번호로 바인딩합니다. 

4. **Listen**

```c
int listen(int sockfd, int backlog);
```

서버 소켓을 passive 모드로 만드는데, 클라이언트가 접근해서 연결을 맺기를 기다립니다. backlog 는 특정 소켓에 연결하기를 기다리는 요청을 담은 큐의 최대 길이를 정의합니다. 만약 큐가 다 찼는데 요청이 오면, 클라이언트는 ECONNREFUSED 와 같은 에러를 받을 수 도 있습니다. 

5. **Accept**

```c
int new_socket= accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

listen 하고 있는 소켓의 큐에 있는 첫번째 요청을 뽑아냅니다. 그 후 연결된 새로운 소켓을 만들고, 그 소켓을 가리키는 새로운 파일 디스크립터를 리턴합니다. 이 시점에서 서버와 클라이언트 사이에 연결은 수립되었고, 데이터를 전송할 준비는 완료되었습니다. 


### Stages for Client

이제 **클라이언트** 쪽에서 준비해야 하는 단계들을 봅시다. 


1. **Socket Connection**

서버 쪽에서 하는 일과 동일하게 동작합니다.

2. **Connect**

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

`connect()` 시스템 콜은 sockfd 에 의해 명시된 소켓에 연결합니다. 서버의 주소와 포트는 addr 에 명시되어 있습니다. 

### 구현

- `server.c`

```c
// Server side C program to demonstrate Socket
// programming
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080
int main(int argc, char const* argv[])
{
	int server_fd, new_socket;
	ssize_t valread;
	struct sockaddr_in address;
	int opt = 1;
	socklen_t addrlen = sizeof(address);
	char buffer[1024] = { 0 };
	char* hello = "Hello from server";

	// Creating socket file descriptor
	if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		perror("socket failed");
		exit(EXIT_FAILURE);
	}

	// Forcefully attaching socket to the port 8080
	if (setsockopt(server_fd, SOL_SOCKET,
				SO_REUSEADDR | SO_REUSEPORT, &opt,
				sizeof(opt))) {
		perror("setsockopt");
		exit(EXIT_FAILURE);
	}
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = INADDR_ANY;
	address.sin_port = htons(PORT);

	// Forcefully attaching socket to the port 8080
	if (bind(server_fd, (struct sockaddr*)&address,
			sizeof(address))
		< 0) {
		perror("bind failed");
		exit(EXIT_FAILURE);
	}
	if (listen(server_fd, 3) < 0) {
		perror("listen");
		exit(EXIT_FAILURE);
	}
	if ((new_socket
		= accept(server_fd, (struct sockaddr*)&address,
				&addrlen))
		< 0) {
		perror("accept");
		exit(EXIT_FAILURE);
	}
	valread = read(new_socket, buffer,
				1024 - 1); // subtract 1 for the null
							// terminator at the end
	printf("%s\n", buffer);
	send(new_socket, hello, strlen(hello), 0);
	printf("Hello message sent\n");

	// closing the connected socket
	close(new_socket);
	// closing the listening socket
	close(server_fd);
	return 0;
}

```

- `client.c`

```c
// Client side C program to demonstrate Socket
// programming
#include <arpa/inet.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>
#define PORT 8080

int main(int argc, char const* argv[])
{
	int status, valread, client_fd;
	struct sockaddr_in serv_addr;
	char* hello = "Hello from client";
	char buffer[1024] = { 0 };
	if ((client_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
		printf("\n Socket creation error \n");
		return -1;
	}

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons(PORT);

	// Convert IPv4 and IPv6 addresses from text to binary
	// form
	if (inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr)
		<= 0) {
		printf(
			"\nInvalid address/ Address not supported \n");
		return -1;
	}

	if ((status
		= connect(client_fd, (struct sockaddr*)&serv_addr,
				sizeof(serv_addr)))
		< 0) {
		printf("\nConnection Failed \n");
		return -1;
	}
	send(client_fd, hello, strlen(hello), 0);
	printf("Hello message sent\n");
	valread = read(client_fd, buffer,
				1024 - 1); // subtract 1 for the null
							// terminator at the end
	printf("%s\n", buffer);

	// closing the connected socket
	close(client_fd);
	return 0;
}

```

실제로 server.c, client.c 를 컴파일하고 실행해봅시다. 이때, client.c 먼저 실행하면 아래와 같이 connection failed 오류가 납니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/63839901-ad1a-4ea8-9312-0fbd885d198b)

서버쪽 소켓을 먼저 생성한 후, 클라이언트가 연결해야 합니다. 아래와 같이 실행하면 정상 동작하는 것을 확인할 수 있습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d332ae82-8530-4f17-9fa3-7ac5fcab17eb)


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/38672bd3-7e56-4ed1-ab92-82db8595842b)

## 소켓은 OSI 7계층 중 어디에 속할까요?

![image](https://github.com/ddoddii/Multicore-GPU-Programming/assets/95014836/4d3b64d1-a859-43a2-9c8f-1115865e1d14)


소켓은 OSI 7계층 중 **세션 레이어**에 속합니다. 소켓은 하나의 컴퓨터와 다른 컴퓨터가 소통하기 위한 창구같은 개념입니다. 네트워크 소켓(network socket)은 컴퓨터 네트워크를 경유하는 프로세스 간 통신의 종착점입니다. 오늘날 컴퓨터 간 통신의 대부분은 인터넷 프로토콜을 기반으로 하고 있으므로, 대부분의 네트워크 소켓은 인터넷 소켓입니다. 네트워크 통신을 위한 프로그램들은 소켓을 생성하고, 이 소켓을 통해서 서로 데이터를 교환합니다. 

## Reference
- How Network Works, Tsutomu Tone 
- https://www.geeksforgeeks.org/socket-programming-cc/
- https://jmvidal.cse.sc.edu/talks/javasockets/sockets.html
- https://en.wikipedia.org/wiki/Network_socket