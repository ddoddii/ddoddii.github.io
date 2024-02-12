+++
author = "Soeun"
title = "OpenAI streaming using websockets"
date = "2024-02-11"
summary = "프로젝트에 OpenAI steam 기능 사용기"
categories = [
    "CS"
]
tags = [
    "OpenAI",
]
slug = "openai-stream"
+++

## OpenAI Streaming

OpenAI 의 응답을 기다리는 것은 시간이 오래 걸립니다. 따라서, OpenAI 에서는 stream 기능을 제공합니다. 아래는 OpenAI API reference 에 나온 Stream 에 대한 소개입니다.

> The OpenAI API provides the ability to stream responses back to a client in order to allow partial results for certain requests. To achieve this, we follow the [Server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events) standard.

아래는 공식문서에서 소개하는 python 을 이용한 stream 을 사용하는 예시입니다.

```python
from openai import OpenAI

client = OpenAI()

stream = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Say this is a test"}],
    stream=True,
)
for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="")
```

그럼 여기서 [Server-sent-events](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events) 란 무엇일까요? 다른 실시간 통신 기술인 websockets 와는 다른점이 무엇일까요?

## SSE, Eventsource, Websockets

### SSE, EventSource

우선, **<span style="background:#fff88f">Server-Sent-Events(SSE)</span>** 는 단방향입니다. 즉, 서버에서 클라이언트로의 **단방향 통신**만 지원합니다. 이는 서버가 클라이언트에게 데이터를 푸쉬할 수 있지만, 클라이언트가 서버로 데이터를 보내는 것은 별도의 요청을 통해야 합니다. 또 SSE 는 **텍스트 데이터**를 전송하는데 최적화되어 있으며, 주로 JSON 형식으로 데이터를 전송합니다. 클라이언트와 서버 간 연결이 끊어졌을 때, SSE 클라이언트는 자동으로 서버에 재연결을 시도합니다. **HTTP 1.1 프로토콜** 위에서 작동합니다.

또한, 모던 자바스크립트 사이트에서는 아래와 같은 설명이 나와있습니다. 여기서 **EventSource** 란 또 어떤 것일까요?

> The [Server-Sent Events](https://html.spec.whatwg.org/multipage/comms.html#the-eventsource-interface) specification describes a built-in class `EventSource`, that keeps connection with the server and allows to receive events from it.

SSE는 서버에서 클라이언트로 실시간 데이터를 전송하는 방법을 정의하는 웹 기술이라면, **<span style="background:#fff88f">EventSource</span>** 는 이 SSE를 구현하기 위해 사용되는 **JavaScript API** 입니다. 아래와 같이 사용할 수 있습니다. 웹 페이지의 JavaScript 에서 이 API 를 사용하여 서버로부터 오는 이벤트를 수신할 수 있습니다. `Eventsoucre` 객체를 생성하여 서버의 특정 URL 에 연결하면, 서버는 그 URL 을 통해 클라이언트에게 데이터를 전송할 수 있습니다.

이 API는 서버로부터 데이터를 수신할 때마다 발생하는 이벤트를 처리하기 위한 메서드를 제공합니다. 예를 들어, `onmessage` 이벤트 핸들러를 사용하여 데이터가 도착할 때마다 호출될 함수를 정의할 수 있습니다.

이렇게, SSE와 EventSource는 함께 작동하여 웹 애플리케이션에서 실시간 데이터 통신을 가능하게 합니다. SSE는 서버 측 기술이며, EventSource는 그 기술을 웹 브라우저에서 사용하기 위한 클라이언트 측 API입니다.

```javascript
var source = new EventSource('/path/to/sse-server');
source.onmessage = function (event) {
	// 서버로부터 메시지를 받을 때 실행될 코드
	console.log('Received data:', event.data);
};
```

## WebSockets

반면, **<span style="background:#fff88f">WebSockets</span>** 는 서버와 클라이언트 간의 **양방향 통신**을 지원합니다. 이를 통해 둘 사이에 실시간 데이터 교환이 가능합니다. 또한 WebSockets은 **텍스트와 이진 데이터** 모두를 전송할 수 있어, 멀티미디어 데이터나 바이너리 데이터의 실시간 전송에 적합합니다. 연결이 한 번 수립되면, 클라이언트와 서버 간에 지속적인 연결이 유지됩니다. WebSockets은 HTTP 프로토콜 위에서 초기 핸드셰이크를 수행한 후, 연결을 업그레이드하여 **별도의 프로토콜**로 통신합니다. 이로 인해 구현이 SSE보다 복잡할 수 있습니다. 클라이언트와 서버 간의 연결을 시작할 때, HTTP 요청을 통해 WebSocket 연결로의 프로토콜 전환(upgrade)이 필요합니다.

| WebSocket                                                    | EventSource                             |
| ------------------------------------------------------------ | --------------------------------------- |
| Bi-directional: both client and server can exchange messages | One-directional: only server sends data |
| Binary and text data                                         | Only text                               |
| WebSocket protocol                                           | Regular HTTP                            |

WebSockets 에 대해 좀 더 자세히 알아봅시다. Websockets 와 일반적인 HTTP request-response 의 차이점은 무엇일까요 ?

Websockets , HTTP 는 둘 다 client-server 소통에 사용되는 프로토콜입니다.

#### HTTP protocol

HTTP 는 클라이언트가 요청을 보내고 서버가 응답을 보내는 단방향성입니다. 사용자가 서버에게 요청을 보낼 때, 이 요청은 HTTP 또는 HTTPS 형태로 가고, 서버는 요청을 받은 후 각 요청에 따른 응답을 전송합니다. 응답이 전송이 끝나면, 연결은 해제됩니다. 따라서 각 HTTP 또는 HTTPS 요청을 할 때마다 새로운 연결을 만들어야 합니다. HTTP 는 TCP 위에서 작동하는 **stateless 프로토콜**입니다. TCP는**connection-oriented 프로토콜**로, 3-way handshake 과 재전송으로 인해 data packet 이 잘 전달되는 것을 보장합니다.

여기서 statelesss 의 개념과 connection-oriented 의 개념을 헷갈리면 안됩니다. HTTP는 **stateless 프로토콜**로, 이는 HTTP 서버가 두 개의 연속된 요청 사이에 어떠한 사용자 상태나 데이터도 기억하지 않는다는 의미입니다. 각 요청은 독립적이며, 서버는 요청 사이에 클라이언트의 상태 정보를 유지하지 않습니다. 클라이언트가 다시 연결하더라도, 서버는 이전 연결과 현재 연결 사이의 관계를 인식하지 못합니다.

TCP 는 **connection-oriented 프로토콜**로, 이는 데이터 교환을 시작하기 전에 클라이언트와 서버 간에 미리 연결을 설정해야 한다는 것을 의미합니다. 연결이 설정되면, 데이터가 전송되는 동안 클라이언트와 서버는 지속적으로 연결 상태를 유지합니다. 이 과정에서 패킷의 순서, 데이터의 무결성, 전송 오류 검출 등을 관리합니다. TCP의 연결 지향적 특성은 데이터가 정확한 순서로, 신뢰성 있게 전송되도록 보장합니다. 연결이 유지되는 동안, 손실된 패킷이나 오류가 발생하면 자동으로 재전송을 시도합니다.

HTTP 는 L7, 즉 어플리케이션 레이어 프로토콜이고, TCP 는 L4, 네트워크 레이어 프로토콜입니다. TCP의 connection-oriented 특성은 네트워크 레벨에서의 신뢰성 있는 데이터 전송을 담당합니다. 반면, HTTP의 stateless 특성은 애플리케이션 레벨에서 각 요청을 독립적으로 처리하는 방식을 설명합니다.

다시 돌아와서, HTTP 는 TCP, SCTP 등 어느 connection-oriented 프로토콜 위에서든 작동할 수 있습니다. 클라이언트가 서버에게 HTTP 요청을 보내면, 클라이언트와 서버 사이에 TCP 연결이 열립니다. 클라언트가 응답을 받으면, TCP 연결을 종료됩니다. 즉, 각 HTTP 요청마다 새로운 TCP 연결을 만듭니다. 만약 클라이언트가 서버에게 10개의 HTTP 요청을 보내면, 서로 다른 10개의 TCP 연결이 생깁니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/c76c5c92-e12f-4b61-8273-cd4dfc729adb)

#### WebSockets

WebSockets은 웹 애플리케이션에서 실시간 양방향 통신을 가능하게 하는 기술로, 기존의 HTTP 프로토콜이 제공하는 단방향, 비연속적인 통신의 한계를 넘어서려는 목적으로 설계되었습니다.

Websockets 는 HTTP 와 동일하게 클라이언트-서버 간 통신할 때 사용되는 프로토콜입니다. 하지만 HTTP 와 다르게 ws://, wss:// 로 시작합니다. 또한, Websockets 는 stateful 한 프로토콜로, 클라이언트와 서버 중 한 쪽이 통신을 끝낼 때까지 연결이 살아있습니다.

클라이언트-서버 간 통신 시나리오를 다시 한 번 봅시다. WebSockets 연결은 HTTP 요청을 통해 초기 핸드셰이크를 시작합니다. 이 과정에서 클라이언트는 서버에게 WebSocket 연결을 요청합니다. 이것은 헤더에 "Connecton:Upgrade" 와 "Upgrade:Websocket" 을 붙임으로써 표시할 수 있습니다. 서버가 이를 수락하면 HTTP 연결이 WebSocket 프로토콜을 사용하는 연속적이고 양방향적인 통신 채널로 업그레이드됩니다.

아래는 웹소켓 연결 요청 예시입니다.

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

서버의 수락 응답 또한 HTTP 로 도착합니다. 정상적인 상태코드는 101(Switching Protocols) 입니다. "Sec-Websocket-key" 헤더를 통해 받은 값에 특정 값을 붙인 후, SHA-1 로 해싱하고, base64로 인코딩한 값을 "Sec-WebSocket-Accept" 헤더에 보냅니다. 이 값을 통해 클라이언트는 정상적인 핸드쉐이킹 과정을 검증합니다.

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

Websockets 도 TCP 위에서 동작하기 때문에, TCP가 제공하는 모든 신뢰성 있는 통신 기능 — 데이터 순서 보장, 오류 검출 및 재전송, 흐름 제어 등 — 을 제공받습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/20c771a4-5974-497e-bcba-855a0ec85f69)

따라서 Websockets 는 실시간 web-application 프로그램 또는 채팅 프로그램에 사용될 수 있습니다. 실시간 웹 응용 프로그램은 백엔드 서버에 의해 지속적으로 전송되는 클라이언트 끝에서 데이터를 보여주기 위해 웹 소켓을 사용합니다. 웹 소켓에서는 이미 열려 있는 동일한 연결로 데이터가 지속적으로 푸시/전송되기 때문에 웹 소켓이 더 빠르고 응용 프로그램 성능이 향상됩니다.

그렇다면 무조건 WebSockets 이 HTTP 보다 좋은 것일까요? 만약, 오래된 데이터를 가져오거나 한번만 데이터를 가져오는 경우에는 Websockets 보다 HTTP 연결을 하는 것이 훨씬 실용적일 수 있습니다.

## 실제 사용기

저는 OpenAI 의 stream 기능을 사용하여 CV 에 대한 개인 질문들을 stream 형태로 받아서 사용자에게 실시간으로 보여지게 했습니다.

실제 prompt 는 더 길게 yml 파일로 작성했지만, 여기서는 간단한 예시만 첨부했습니다.

```python
from typing import AsyncGenerator, NoReturn

import uvicorn
from dotenv import load_dotenv
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse
from openai import AsyncOpenAI

load_dotenv()

client = AsyncOpenAI()
app = FastAPI()


with open("index.html") as f:
	html = f.read()

async def get_ai_response(message: str) -> AsyncGenerator[str, None]:
	"""
	OpenAI Response
	"""
	response = await client.chat.completions.create(
		model="gpt-3.5-turbo",
		messages=[
			{
				"role": "system",
				"content": (
					"You are a brilliant interviewer. "
					"The Question and Assessment point is provided below."
					),
			},
			{
				"role": "user",
				"content": message,
			},
		],
		stream=True,
	)

	all_content = ""
	async for chunk in response:
		content = chunk.choices[0].delta.content
			if content:
				all_content += content
				yield all_content

@app.get("/")
async def web_app() -> HTMLResponse:
	"""
	Web App
	"""
	return HTMLResponse(html)

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket) -> NoReturn:
	"""
	Websocket for AI responses
	"""
	await websocket.accept()
	while True:
		message = await websocket.receive_text()
		async for text in get_ai_response(message):
			await websocket.send_text(text)

if __name__ == "__main__":
	uvicorn.run(
	"main:app",
	host="0.0.0.0",
	port=8000,
	log_level="debug",
	reload=True,
	)
```

### 포인트1. AsyncGenerator 사용

여기서 `get-ai-response` 의 리턴 타입으로 `AsyncGenerator` 를 사용했습니다. python 에는 그냥 `generator` 도 있는데, `AsyncGenerator` 를 사용한 이유에 대해 조금 자세히 설명해보고자 합니다.

우선,**generator** 는 yield 표현을 포함하는 함수입니다. generator 를 사용하면, 생성된 값을 산출하기 위해 순회할 수 있는 generator 반복기가 반환됩니다. 문제는 기존 generator는yield 된 값들을 기다릴 수 없기 때문에 비동기 프로그램에 적합하지 않다는 것입니다.

> A Python generator is any function containing one or more yield expressions

> generator: A function which returns a generator iterator. It looks like a normal function except that it contains yield expressions for producing a series of values usable in a for-loop or that can be retrieved one at a time with the next() function.

따라서, 코루틴과 yield value 를 사용하여 **asynchronous generator** 를 정의할 수 있습니다. asynchronous generator는 async for 표현을 사용해서 각 yielded value 를 자동으로 await 하며 순회할 수 있습니다. asynchronous generator 는 yield 표현식을 사용하는 코루틴입니다. generator 함수와는 다르게, 코루틴은 다른 코루틴 또는 태스크를 기다릴 수 있고, 스케쥴될 수 있습니다.

> asynchronous generator: A function which returns an asynchronous generator iterator. It looks like a coroutine function defined with async def except that it contains yield expressions for producing a series of values usable in an async for loop.

전통적인 generator와 비슷하게, asynchronous generator는 asynchronous generator iterator 를 만들 수 있습니다. asynchronous generator iterator 는 `anext()` 를 사용해서 순회될 수 있습니다.

그럼 두가지의 차이점은 무엇일까요?

- Classical generator
  - 파이썬 프로그램 어디에서나 사용됩니다.
  - `next()` function 을 사용하여 다음 스텝으로 넘어갑니다.
  - for loop 을 사용하여 순회합니다.
  - 값을 yield 합니다.
- Asynchronous generator
  - asyncio 프로그램에서 사용됩니다.
  - `anext()` function 을 사용하여 다음 스텝으로 넘어갑니다.
  - async for loop 을 사용하여 순회합니다.
  - awaitable 을 yield 합니다.

자, 그러면 다시 돌아와서 위의 코드에서 asynchronous generator 를 사용한 이유에 대해 살펴봅시다. 우선 api 를 요청하고 응답을 기다리는 시간은 CPU 를 거의 사용하지 않는 I/O bound 작업입니다. 따라서 비동기식으로 처리하는 것이 합리적입니다.

기존의 generator 는 동기적 작업을 위해 사용됩니다. generator 를 사용하는 동안, yield 에서 반환되는 작업이 블로킹 작업인 경우, 해당 작업이 완료될 때까지 전체 프로그램의 실행이 중단됩ㄴ다.

반면 asynchronous generator 는 비동기적 작업, 특히 I/O 바운드 작업(I/O-bound operations)을 처리할 때 사용됩니다. `async for` 루프 내에서 `await`를 사용하여 비동기적으로 값을 하나씩 처리할 수 있습니다. `AsyncGenerator`를 사용하면 비동기 작업을 기다리는 동안 프로그램의 다른 부분이 실행될 수 있어, 논블로킹(non-blocking) 동작을 구현할 수 있습니다. 이는 특히 웹 애플리케이션, 네트워크 요청 처리, 고성능 API 호출 등에서 중요합니다.

### 포인트2. Websocket 사용

FastAPI 에서는 [websocket API](https://fastapi.tiangolo.com/advanced/websockets/)를 제공해줍니다. 우선, `await websocket.accept()` 를 사용하여 서버가 클라이언트의 요청을 비동기적으로 수락하도록 합니다. 이것은 클라이언트와 서버 간의 websocket 연결이 성공적으로 수립되었음을 의미합니다.

그 다음, `while True` 루프 속에서, `async for` 루프 를 사용하여 `get_ai_response` 함수를 통해 비동기적으로 생성된 AI의 응답을 순회합니다. `await websocket.send_text(text)`는 생성된 응답 텍스트를 클라이언트에게 비동기적으로 전송합니다. 이렇게 서버는 클라이언트의 각 요청에 대해 실시간으로 응답을 생성하고 전송할 수 있습니다.

## Reference

- https://ko.javascript.info/server-sent-events
- https://ko.wikipedia.org/wiki/%EC%9B%B9%EC%86%8C%EC%BC%93
- https://www.geeksforgeeks.org/what-is-web-socket-and-how-it-is-different-from-the-http/
- https://tecoble.techcourse.co.kr/post/2020-09-20-websocket/
- https://superfastpython.com/asynchronous-generators-in-python/
- https://fastapi.tiangolo.com/advanced/websockets/
