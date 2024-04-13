+++
author = "Soeun"
title = "Locust 부하테스트를 통해 알아본 적절한 pool size 설정하기"
date = "2024-01-30"
summary = "커넥션 풀 사이즈는 무엇이며 어떻게 설정할까?"
categories = [
    "CS"
]
tags = [
    "locust"
]
slug = "pool-size"
+++

## Locust 를 이용한 부하 테스트

Chat your Interview 에서 동시에 몇명까지 접속해서 로그인/회원가입 기능을 사용할 수 있는지 테스트하기 위해 [Locust](https://locust.io/) 부하 테스트를 사용했습니다. Locust 는 파이썬 으로 테스트코드를 작성할 수 있기 때문에 기존 소스 코드와 함께 사용하기 쉬워 선택했습니다.

### 첫번째 테스트

- 테스트 조건 
	- Number of users : 30
	- Spawn rate : 3

처음 부하테스트를 한 결과는 처참했습니다. 


![KakaoTalk_Photo_2024-01-10-00-00-32](https://github.com/ddoddii/resume-ai-chat/assets/95014836/10e15c5a-e3d2-47d3-9e6f-0cae79315bc6)

30명에서 못 버티고 계속 fail 이 일어났다. 저는 AWS 에서 docker 를 이용해서 실행중이었으므로, docker container logs 를 찍어보니 다음과 같은 오류가 발생했습니다. 


```text
raise exc.TimeoutError(sqlalchemy.exc.TimeoutError: QueuePool limit of size 5 overflow 10 reached, connection timed out, timeout 30.00)
```

원인을 구글링해보니, 2가지로 추릴 수 있었습니다. 

1. SQLalchemy 에서 session을 생성하고, close 하지 않은 것. 이렇게 되면 session 은 계속 내 연결을 잡아먹고 pool 에 리턴하지 않는다. 
2.  SQLalchemy Engine 을 만들 때 애초에 pool_size, max_overflow 한계를 작게 잡은 것입니다. 

1번을 확인해보니, 저는 애초에 db_dependency 를 주입할 때 아래와 같이 만들어서 주입하므로 session 이 종료되지 않았을 걱정은 없었습니다. 

```python
def get_db():  
    db = SessionLocal()  
    try:  
        yield db  
    finally:  
        db.close()  
  
  
db_dependency = Annotated[Session, Depends(get_db)]
```

그렇다면 2번이 원인이라는 것인데, 여기서 우선 **Connection Pooling**, **pool_size**, **max_overflow** 가 어떤 것을 뜻하는지 알아봅시다. 

### Connection Pooling

커넥션 풀이란 미래에 데이터베이스에 요청을 보낼 때를 대비해서 데이터베이스 연결을 만들어 놓은 것입니다. 커넥션 풀 은 DB 에 요청을 보낼 때 성능을 높이기 위해 사용합니다. 커넥션 풀이 없다면, DB 에 요청을 보낼 때마다 새로운 연결을 만들어야 할 것입니다. 

하지만 실제로 OS에서 감당할 수 있는 현실적인 커넥션 수가 있습니다. 각 데이터베이스 연결에는 추가 메모리도 사용합니다. 풀 크기를 제한하면 프로그램의 버그나 서버에 대한 악의적인 공격으로부터 데이터베이스를 보호하는 데 도움이 됩니다. 너무 많은 연결을 사용하거나 너무 많은 메모리를 사용하면 데이터베이스 서버를 정지시킬 수 있습니다. 따라서 커넥션 풀에는 만들 수 있는 커넥션의 제한을 둬야 합니다.

일반적인 상황에서는 각 연결에서 사용하는 추가 메모리가 큰 문제가 되지는 않지만 동시에 사용할 것(concurrently)으로 생각되는 최대 개수로 제한하는 것이 가장 좋습니다.

**pool_size**

pool_size 는 pool 내에 몇개의 연결을 유지할 것인지를 정의합니다. 만약 pool 내에 있는 모든 연결이 사용중이고 새로운 요청이 들어오면, 그 요청은 연결이 가능해질 때까지 기다려야 합니다. 

SqlAlchemy 에서 pool_size 의 기본값은 5입니다. StackOverFlow 의 답변들을 찾아보니 보통 20으로 설정하는 것이 적정하다고 나와 있습니다. 

**max_overflow**

max_overflow 는 만약 pool 내에 있는 모든 연결이 사용 중일 때 pool_size 를 능가해서 새롭게 만들 수 있는 연결의 수입니다. pool_size 만큼의 연결이 사용중이면, 추가 연결들 (max_overflow 까지) 이 만들어지고 추가적으로 로드를 처리할 수 있습니다. 만약 로드가 감소하면, 이 max_overflow 에 해당하는 연결들이 가장 먼저 종료됩니다. 


**그렇다면, 무한정으로 pool_size, max_overflow 를 늘리는 것이 모든 것을 해결해줄 수 있는 은총알이 될 수 있나?** 하면 틀린 생각입니다. 모두 trade-off 가 있습니다. 

우선, 리소스가 부족할 수 있습니다. 각 연결은 DB 서버, 어플리케이션 서버에 리소스를 소비합니다. 연결이 많을 수 록 서버에 많은 부담이 가고, 이것은 성능 저하로 이어질 수 있습니다. 

DB에서 동시성 문제도 발생할 수 있습니다. DB에서는 다른 동시에 접속하는 연결들을 어느 정도까지는 효율적으로 처리합니다. 그 이상으로는, DB lock 과 병목현상이 발생할 수 있습니다. 

따라서 내가 가지고 있는 리소스를 고려해서 최대로 열어둘 수 있는 pool_size 가 무엇인지 결정해야 합니다. 또한, 쿼리를 최적화하는 것, 필요없는 데이터베이스 콜들을 줄이는 것, 결과를 캐싱하는 것이 단순히 pool_size 를 늘리는 것보다 더 나은 해결책이 될 수 있습니다. 

----

일단 저는 session 이 제대로 종료되고 쓸데없이 DB 를 부르는 콜들이 없는지도 확인했으므로, 기본값인 pool_size = 5 에서 20 으로 변경하고, max_overflow 도 20으로 변경했습니다. 

```python
from sqlalchemy import create_engine

engine = create_engine(  
    SQLALCEMY_DATABASE_URL,  
    connect_args={"check_same_thread": False},  
    pool_size=20,  
    max_overflow=20,  
)
```



### 두번째 테스트 

- 테스트 조건 
	- Number of users : 30
	- Spawn rate : 3


![image](https://github.com/ddoddii/resume-ai-chat/assets/95014836/cabeedc6-c009-40d6-a6b0-79627117f2c4)



![total_requests_per_second_1704812012](https://github.com/ddoddii/resume-ai-chat/assets/95014836/8a2d2ea1-bd3c-4d7e-83a4-55590dcce5f8)

두번째 테스트를 보면 30명이 동시 접속할 때 failure 는 1.4%만 일어나는 것으로 확인할 수 있습니다. 

이렇게 동시성 테스트도 해보고, 어떤 조건들을 튜닝해야 할 지 알아보았습니다. 



## Reference
- https://docs.sqlalchemy.org/en/20/core/pooling.html
- https://stackoverflow.com/questions/38534203/why-limit-db-connection-pool-size-in-sqlalchemy?rq=3