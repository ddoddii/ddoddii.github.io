+++
author = "Soeun"
title = "python 에서의 Async 개념"
date = "2023-12-21"
summary = "python 공식 문서를 참고한 async, await, coroutine, task 개념 정리"
categories = [
    "CS"
]
tags = [
    "python"
]
slug = "python-asyncio"
+++

## Async 

우선 예시를 통해 synchronous, asynchronous 를 비교해보자. 

```python 
func1()
func2()
func3()
```

위의 코드가 **synchronous** 라고 하면, func1 -> func2 -> func3 순서대로 호출이 된다. 다음 함수는 전의 함수가 끝나야지만 호출이 된다. 

Multi-threading 이면, 3개의 thread 를 만들고 각각 쓰레드에서 함수들을 실행시킬 것이다. 

**Asynchronous** 에서는, func1 이 DB 로부터 데이터를 가져온다고 하자. 그럼 func1 이 await 하거나, 생산적이지 않은 시간이 있을 것이다. 이 시간을 이용해서 func2 를 실행시킨다. 

Asynchronous 는 single-threaded 환경에서 여러 태스크 들을 동시에 실행(concurrent execution) 할 수 있도록 하는 프로그래밍 패러다임이다. 즉 async 는 멀티쓰레딩, 멀티프로세싱도 아니고 동시성 프로그래밍이다.  

파이썬에서는 `asyncio` 라이브러리를 이용하여 구현할 수 있다. asyncio 에서는 async / await 구문을 이용하여 동시성 코드를 작성할 수 있다. asynco 는 고성능 네트워크 및 웹 서버, 데이터베이스 연결 라이브러리, 분산 작업 큐 등을 제공하는 여러 파이썬 비동기 프레임워크의 기반으로 사용된다.  asyncio 는 종종 IO 병목이면서 고수준의 **구조화된** 네트워크 코드에 가장 적합하다. 

`asyncio` 는 다음과 같은 고수준 API 를 제공한다. 

- 파이썬 코루틴들을 동시에 실행하고 실행을 완전히 제어할 수 있다.
- 네트워크 IO 와 IPC 를 수행한다.
- 자식 프로세스를 제어한다.
- 큐를 통해 작업을 분산한다.
- 동시성 코드를 동기화 한다. 


```python 
import asyncio

async def main():
	task = asyncio.create_task(other_function())
	print("A")
	await asyncio.sleep(1)
	print("B")
	await task

async def other_function():
	print("1")
	await asyncio.sleep(2)
	print("2")

asyncio.run(main())

```

결과는 아래와 같다. 
```text
A
1
B
2
```

`asyncio.create_task()` 를 이용하여 task 를 만든다. 이 task 는 main 이 실행되는 동안 비어있는 시간 (`asyncio.sleep(1)`) 의 시간 동안 실행한다. 따라서 A 를 프린트하고, 비어있는 1초 동안 `other_function()` 을 실행해서 1이 프린트 된다. 1초가 끝나면 다시 main 으로 돌아와서 B 가 프린트 된다. 그 후 다시 `await task` 를 하므로, task 가 끝날 때까지, 즉 `asyncio.sleep(2)` 를 하고 2가 프린트 될때까지 기다린 후 종료 된다. 

이 **asynco api** 에 대해 좀 더 알아보자. 

## Coroutine

코루틴은 async / await 를 사용해서 선언된다. 즉 `async def` 로 선언된 함수는 코루틴이다. 
```python
import asyncio

async def main():
    print('hello')
	await asyncio.sleep(1)
	print('world')

asyncio.run(main())

# 출력 결과
hello
world
```

여기서 `main()` 만 실행하면, coroutine object 자체가 출력된다. 왜냐면 main() 만 호출하면, 코루틴을 호출하는 것이지, 실행되도록 예약하는 것은 아니기 때문이다. 

```python 
>>> main()
<coroutine object main at 0x1053bb7c8>
```

코루틴을 직접적으로 실행하려면, 다음과 같은 방법을 따라야 한다. 
- 최상위 진입점 `main()` 함수를 실행하는 `asyncio.run()` 함수
- 코루틴을 기다리기. 
  
  아래 코드는 1초를 기다린 후 "hello" 를 프린트 하고, 2초를 기다린 후 "world" 를 프린트한다.  출력으로 보면 총 3초가 걸렸음을 알 수 있다. 

	```python
	import asyncio
	import time
	
	async def say_after(delay, what):
		await asyncio.sleep(delay)
		print(what)
	
	async def main():
		print(f"started at {time.strftime('%X')}")
	
		await say_after(1, 'hello')
		await say_after(2, 'world')
	
		print(f"finished at {time.strftime('%X')}")
	
	asyncio.run(main())
	```
	
	예상 출력 
	```python
	started at 17:13:52
	hello
	world
	finished at 17:13:55
	```

- 코루틴을 asyncio 태스크로 동시에 실행하는 `asyncio.create_task()` 함수 
  
  위의 예시를 조금 수정해서 `say_after` 코루틴을 동시에 실행해보자. 출력을 보면, 위의 결과보다 1초 빠른 2초가 걸렸음을 알 수 있다. 
  
	```python
	async def main():
	    task1 = asyncio.create_task(
	        say_after(1, 'hello'))
	
	    task2 = asyncio.create_task(
	        say_after(2, 'world'))
	
	    print(f"started at {time.strftime('%X')}")
	
	    # Wait until both tasks are completed (should take
	    # around 2 seconds.)
	    await task1
	    await task2
	
	    print(f"finished at {time.strftime('%X')}")
	```

	예상 출력
	```python
	started at 17:14:32
	hello
	world
	finished at 17:14:34
	```

	위의 코드는 `asyncio.TaskGroup` 을 사용하여 아래와 같이 나타낼 수도 있다. 
	```python
	async def main():
	    async with asyncio.TaskGroup() as tg:
	        task1 = tg.create_task(
	            say_after(1, 'hello'))
	
	        task2 = tg.create_task(
	            say_after(2, 'world'))
	
	        print(f"started at {time.strftime('%X')}")
	
	    # The await is implicit when the context manager exits.
	
	    print(f"finished at {time.strftime('%X')}")
	```

## Awaitable

객체가 `await` 표현식에서 사용될 수 있을 때 awaitable 객체 라고 한다. awaitable 객체에는 3가지 유형 있다 : coroutine, task, future

### coroutine

파이썬 코루틴 은 awaitable 이므로, 다른 코루틴에서 기다릴 수 있다. 아래 코드를 보자. `nested()` 는 `async def` 를 통해 선언되었으므로 코루틴이다. `main()` 내에서 `nested()` 를 호출하면, 아무 일도 일어나지 않는다. 왜냐하면 이때 직접 실행하는 것이 아니라 단순히 코루틴 객체를 만든다. 코루틴은 `await` 을 통해서 미래에 실행되기를 기다릴 뿐이다. 

반면 아래에서 `print(await nested())` 에서는 42가 출력된다. `await` 는 코루틴을 실행하기 위한 예약어이다. 이 예약어 없이는 코루틴은 실행되지 않는다. 

```python
import asyncio

async def nested():
    return 42

async def main():
    # Nothing happens if we just call "nested()".
    # A coroutine object is created but not awaited,
    # so it *won't run at all*.
    nested()

    # Let's do it differently now and await it:
    print(await nested())  # will print "42".

asyncio.run(main())
```

여기서 2가지 정의를 기억해야 한다.
- 코루틴 함수 : `async def` 함수
- 코루틴 객체 : 코루틴 함수를 호출하여 반환된 객체


### Task

Task 는 코루틴을 동시에 예약하는데 사용된다.
코루틴이 [`asyncio.create_task()`](https://docs.python.org/ko/3/library/asyncio-task.html#asyncio.create_task "asyncio.create_task")와 같은 함수를 사용하여 태스크로 싸일 때 코루틴은 곧 실행되도록 자동으로 예약된다. 


아래 코드에서는 `asyncio.create_task()` 를 통해 태스크를 만들었다. 이것은 2가지 역할을 하는데, (1) 코루틴이 event loop 에 실행될 수 있도록 스케쥴링 , (2) Task 객체 반환 이다. 

```python
import asyncio

async def nested():
    return 42

async def main():
    # Schedule nested() to run soon concurrently
    # with "main()".
    task = asyncio.create_task(nested())

    # "task" can now be used to cancel "nested()", or
    # can simply be awaited to wait until it is complete:
    result = await task
    print(result)

asyncio.run(main())
# 출력 : 42
```

####  Task 만들기

`asyncio.create_task(coro, *, name=None, context=None)`

- coro : 코루틴을 Task 로 감싸고 실행을 예약한다. 
- name : name 이 None 이 아니면, `Task.set_name()` 을 사용하여 태스크의 이름으로 설정된다. 
- context : coro 가 실행된 특정한 context 를 지정할 수 있다. [`get_running_loop()`](https://docs.python.org/ko/3/library/asyncio-eventloop.html#asyncio.get_running_loop "asyncio.get_running_loop")에 의해 반환된 루프에서 태스크가 실행되고, 현재 스레드에 실행 중인 루프가 없으면 [`RuntimeError`](https://docs.python.org/ko/3/library/exceptions.html#RuntimeError "RuntimeError")가 발생한다.  (python 3.11 추가)

#### Task Groups

Task group 는 그룹 내에 있는 태스크가 모두 실행될 수 있게 기다리는 신뢰할 수 있는 방법이다. 

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(some_coro(...))
        task2 = tg.create_task(another_coro(...))
    print(f"Both tasks have completed now: {task1.result()}, {task2.result()}")
```

`async with` 는 Task group 내에 있는 모든 태스크들이 실행될 때까지 기다린다. 기다리는 동안 새로운 태스크가 Task group 에 추가될 수 있다. 마지막 태스크가 실행되고 `async with`  블럭이 끝나면, 그룹에 더 이상 새로운 태스크가 추가될 수 없다. 

만약 그룹 내에 있는 태스크 중에 [`asyncio.CancelledError`](https://docs.python.org/ko/3/library/asyncio-exceptions.html#asyncio.CancelledError "asyncio.CancelledError") 말고 다른 에러로 인해 멈추면, 그룹 내에 남아있는 다른 태스크들도 취소된다. 

### Future
Future  비동기 연산의 **최종 결과**를 나타내는 특별한 **저수준** awaitable 객체이다. 일반적인 응용 프로그램 수준에서 future 객체를 만들 필요는 없다. 예시를 통해 Task 와 future 를 비교해보자. 

내가 음식점에 가서 주문을 했다. 요리사는 아직 주문을 만드는 것을 시작하지 않았지만, 점원은 나에게 토큰을 주었다. 이 토큰은 음식이 아니고 먹을 수도 없지만, 이것은 음식이 준비될 것이고 나에게 줄 것 이라는 약속이다. 나는 이 토큰을 보고 음식이 다 준비되었는지를 알 수 있다.

**Future** 는 마치 이 토큰과 같다. 내가 프로그램에게 시간이 조금 걸리는 작업을 실행하라고 명령했을 때, 프로그램은 Future 를 준다. Future 는 내가 기다리는 결과를 갖고 있다는 약속이지, 결과 자체는 아니다. 나는 Future 를 보고 결과가 끝났는지를 알 수 있다. 

반면 Task 도 Future 과 비슷하지만, 조금 더 앞선 단계이다. 음식점에 요리사의 행동을 보여주는 스크린이 있다고 하자. 요리사는 단순히 음식을 약속하는 것이 아니라, 실제로 재료를 썰고 물을 끓이며 요리를 하고 있다. 

**Task** 는 내가 요청한 결과를 주기 위해서 실제 실행도 같이 하고 있는 단계이다. 

## Sleep

`coroutine asyncio.sleep(delay, result=None)`
- delay 초 동안 블록한다.
- result 가 제공되면, 코루틴이 완료될 때 호출자에게 반환된다.
- `sleep()` 는 항상 현재 태스크를 일시 중단해서 다른 태스크를 실행할 수 있도록 한다. 


## 동시에 태스크 실행하기 
`awaitable asyncio.gather(*aws, return_exceptions = False)`

- aws 시퀀스에 있는 모든 awaitable 객체를 동시에 실행한다. 
- aws 에 있는 awaitable 이 코루틴이면 자동으로 태스크로 예약된다. 
- 모든 awaitable 이 성공적으로 완료되면, 결과는 반환된 값들이 합쳐진 리스트이다. 결괏값의 순서는 aws에 있는 어웨이터블의 순서와 일치한다.
- return_exceptions가 `False`(기본값)면, 첫 번째 발생한 예외가 `gather()`를 기다리는 태스크로 즉시 전파된다. aws 시퀀스의 다른 어웨이터블은 **취소되지 않고** 계속 실행된다. 
  return_exceptions가 `True`면, 예외는 성공적인 결과처럼 처리되고, 결과 리스트에 집계된다.

아래 코드를 보자.
```python
import asyncio  
  
async def factorial(name, number):  
    f = 1  
    for i in range(2, number + 1):  
        print(f"Task {name}: Compute factorial({number}), currently i={i}...")  
        await asyncio.sleep(1)  
        f *= i  
    print(f"Task {name}: factorial({number}) = {f}")  
    return f  
  
async def main():  
    # Schedule three calls *concurrently*:  
    L = await asyncio.gather(  
        factorial("A", 2),  
        factorial("B", 3),  
        factorial("C", 4),  
    )  
    print(f"asynco gather output: {L}") 
  
  
asyncio.run(main())
```

실행 결과
```text
Task A: Compute factorial(2), currently i=2...
Task B: Compute factorial(3), currently i=2...
Task C: Compute factorial(4), currently i=2...
Task A: factorial(2) = 2
Task B: Compute factorial(3), currently i=3...
Task C: Compute factorial(4), currently i=3...
Task B: factorial(3) = 6
Task C: Compute factorial(4), currently i=4...
Task C: factorial(4) = 24
asynco gather output: [2, 6, 24]
```


## 취소로부터 보호하기
`awaitable asyncio.shield(aw)`
- awaitable 객체를 취소로부터 보호한다. 
- aw 가 코루틴이면 자동으로 태스크로 예약된다. 

```python
task = asyncio.create_task(something())
res = await asyncio.shield(task)
```


```python
res = await something()
```

위의 코드와 아래 코드의 결과 동일하다. 하지만 위의 코드는 외부에서 태스크를 취소하더라도, shield 로 감싼 태스크는 취소되지 않는다. 아래 코드에서는 something() 함수를 직접 호출하고 그 결과를 기다린다. 이 경우에 something() 함수가 실행되는 동안 외부에서 코루틴이 취소되면 something() 함수도 같이 취소될 수 있다. 

## 시간 제한

`asyncio.timeout(delay)`

이 함수는 기다리는 데 걸리는 시간을 제한하는 [asynchronous context manager](https://docs.python.org/ko/3/reference/datamodel.html#async-context-managers) 를 리턴한다. 

delay 는 None 이거나 기다리는 시간을 나타내는 float/int (초) 일 수 있다. context manager 생성 이후에도 [`Timeout.reschedule()`](https://docs.python.org/ko/3/library/asyncio-task.html#asyncio.Timeout.reschedule "asyncio.Timeout.reschedule") 를 이용하여 다시 스케쥴링 할 수 있다. 

```python
async def main():
    async with asyncio.timeout(10):
        await long_running_task()
```

위의 코드에서 만약 `long_running_task()` 가 10초 이상 걸리면, context manager 가 현재 태스크를 취소하고 내부에서 [`asyncio.CancelledError`](https://docs.python.org/ko/3/library/asyncio-exceptions.html#asyncio.CancelledError "asyncio.CancelledError") 를 발생시키고, 이것은  [`TimeoutError`](https://docs.python.org/ko/3/library/exceptions.html#TimeoutError "TimeoutError") 형태로 나타난다. 이 에러는 try-except 를 이용해서 처리할 수 있다.

```python
async def main():
    try:
        async with asyncio.timeout(10):
            await long_running_task()
    except TimeoutError:
        print("The long operation timed out, but we've handled it.")

    print("This statement will run regardless.")
```

다시 스케쥴링 하는 것은 아래와 같이 할 수 있다. 

```python
async def main():
    try:
        # We do not know the timeout when starting, so we pass ``None``.
        async with asyncio.timeout(None) as cm:
            # We know the timeout now, so we reschedule it.
            new_deadline = get_running_loop().time() + 10
            cm.reschedule(new_deadline)

            await long_running_task()
    except TimeoutError:
        pass

    if cm.expired():
        print("Looks like we haven't finished on time.")
```

`coroutine asyncio.wait_for(aw,timeout)`
- aw  awaitable 이 제한된 시간 내에 완료될 때까지 기다린다. 
- aw 가 코루틴이면 자동으로 태스크로 예약된다. 

```python
async def eternity():
    # Sleep for one hour
    await asyncio.sleep(3600)
    print('yay!')

async def main():
    # Wait for at most 1 second
    try:
        await asyncio.wait_for(eternity(), timeout=1.0)
    except TimeoutError:
        print('timeout!')

asyncio.run(main())

# Expected output:
#
#     timeout!
```

## 실제 적용

async , await 를 공부한 이유는 openai api 를 사용할 때 에러가 나지 않게끔 async , await 를 쓴 함수를 봐서이다. 아래는 실제 코드이다. 

```python
async def _request_with_retry(retry=3):  
    for _ in range(retry):  
        try:            
	        response = await openai.ChatCompletion.acreate(  
                model=self.config["model_name"],  
                messages=self.config["messages"],  
                max_tokens=self.config["max_tokens"],  
                temperature=self.config["temperature"],  
                top_p=self.config["top_p"],  
                request_timeout=self.config["request_timeout"],  
            )  
            return response  
        except openai.error.RateLimitError:  
            print("Rate limit error, waiting for 40 second...")  
            await asyncio.sleep(40)  
        except openai.error.APIError:  
            print("API error, waiting for 1 second...")  
            await asyncio.sleep(1)  
        except openai.error.Timeout:  
            print("Timeout error, waiting for 1 second...")  
            await asyncio.sleep(1)  
        except openai.error.ServiceUnavailableError:  
            print("Service unavailable error, waiting for 3 second...") 
            await asyncio.sleep(3)  
        except openai.error.APIConnectionError:  
            print("API Connection error, waiting for 3 second...")  
            await asyncio.sleep(3)  
  
    return None  
  
# async_responses = [  
#     _request_with_retry(messages)  
#     for messages in messages_list  
# ]  
ret = await _request_with_retry()  
return ret
```

`_request_with_retry` 는 `async def`   로 정의되었으므로 코루틴이다. 이 함수는 openAI API 에게 retry (3번) 동안 request 를 보낸다. try-except 문을 사용해서 에러를 처리하고 있다. try 문 안에서 실제 요청을 보내는데, `await openai.ChatCompletion.acreate()` 부분이다. `await` 예약어는 `openai.ChatCompletion.acreate()` 가 코루틴이기 때문에 사용되었다. 실제로 `openai.ChatCompletion.acreate()` 는 아래와 같이 정의되어 있다. 

```python
@classmethod  
async def acreate(cls, *args, **kwargs):  
    """  
    Creates a new chat completion for the provided messages and parameters.  
    See https://platform.openai.com/docs/api-reference/chat-completions/create    for a list of valid parameters.    """ 
    start = time.time()  
    timeout = kwargs.pop("timeout", None)  
  
    while True:  
        try:            
	        return await super().acreate(*args, **kwargs)  
        except TryAgain as e:  
            if timeout is not None and time.time() > start + timeout:  
                raise  
            util.log_info("Waiting for model to warm up", error=e)
```

except 부분에는 일어날 수 있는 에러들이 적혀 있다. (rate limit errors, API errors, timeout errors, service unavailable errors, or API connection errors) 에러가 나타나면 `await asyncio.sleep(1)` 을 통해 1초 동안 블록한다.  그 후 retry 의 range 에 따라 다음 request 를 보내기를 시도한다. 



## Reference
- https://docs.python.org/ko/3/library/asyncio.html
- https://docs.python.org/ko/3/library/asyncio-task.html
