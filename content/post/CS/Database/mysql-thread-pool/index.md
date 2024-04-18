+++
author = "Soeun"
title = "MySQL 서버에서 쓰레드풀은 어떤 역할을 할까?"
date = "2024-04-10"
summary = "클라이언트의 커넥션 풀 - MySQL 의 쓰레드 풀의 관계"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
+++

MySQL **쓰레드 풀**은, MySQL 엔터프라이즈 에디션에 있는 쓰레드 핸들링 메카니즘입니다. 서버 플러그인이지만, 기본값으로 활성화되어 있지는 않습니다. 

MySQL은 기본값으로 **"one-thread-per-connection"** 을 채택하고 있습니다. 사용자가 데이터베이스 서버에 연결하면, 커넥션을 위해 OS의 쓰레드가 생성됩니다. 이 쓰레드는 유저로부터 요청된 모든 쿼리들을 실행하고, 유저가 연결을 끊기 전까지 결과를 전송합니다. 하지만 더 많은 사용자들이 MySQL 서버에 연결을 맺으면, 점점 더 많은 쓰레드가 생성되고 병렬적으로 실행됩니다. 결국에는, 쓰레드를 새롭게 생성하는 것에 대한 한계가 올 것입니다. 동시적 요청이 많은 환경에선, 연결이 증가할 수록 서버의 성능은 저하될 것입니다. 

쓰레드는 OS 스케쥴러가 제공한 타임슬라이스 동안, 혹은 다른 요청을 기다리지 전까지 명령어를 실행합니다. 이때 기다려야 하는 경우는 아래의 경우들이 있을 수 있습니다 : **뮤텍스, 데이터베이스 오브젝트 락, IO**. 쓰레드의 수가 계속 증가하면, CPU 캐시도 꽉 찰 수 있습니다.

MySQL 의 쓰레드풀은 여기에 대한 해결책을 제공합니다. 

## MySQL Thread Pool

MySQL은 유저의 커넥션과 쓰레드들을 분리시킵니다. 이전의 "one-thread-per-connection" 모델과 달리, 유저 커넥션 마다 하나의 쓰레드를 할당하지 않습니다. 대신, 쓰레드 풀은 쓰레드 그룹으로 이루어져 있고, 기본값으로 16개의 쓰레드 그룹이 있습니다. 유저 커넥션은 Round Robin 방식으로 쓰레드 그룹에 할당됩니다. 각 쓰레드 그룹은 유저 커넥션들을 관리합니다. 각 쓰레드 그룹 내에서, 하나 이상의 쓰레드가 그 쓰레드 그룹에 할당된 유저 커넥션으로부터 받은 쿼리들을 실행합니다. 

기본적으로, 쓰레드 풀은 쓰레드 그룹 내에서 미리 설정된 수의 쓰레드만 실행 중임을 보장하기 위해 만들어졌습니다. 하지만 최적의 성능을 달성하기 위해서는, 쓰레드 풀은 쓰레드 그룹 내에서 더 많은 쓰레드들이 실행할 수 있도록 허용해야 합니다. 

아래는 MySQL 클라이언트와 MySQL 서버가 **"Thread Pool"** 모델을 사용해 연결한 모식도입니다. 


![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d466f27e-a95f-4550-a1c7-cde30a3789d6)

- Client : MySQL 클라이언트는 MySQL 서버와 클라이언트-서버 프로토콜을 이용해 소통하는 CLI 혹은 어플리케이션 API 입니다.
- Connection Requests : 클라이언트로부터 온 커넥션 요청은 MySQL 서버로 보내집니다.
- Receiver Thread : 들어오는 커넥션 요청들은 큐에 넣어지고, receiver thread 가 하나씩 처리합니다. Receiver thread 는 RR 을 이용해 요청에 쓰레드 그룹을 할당합니다. 
- Query Worker Threads : 쓰레드 그룹 내에 있는 쓰레드들로, 사용자의 쿼리를 실행합니다.
- THD : 각 유저 커넥션을 위해 생성된 쓰레드 컨텍스트 자료구조입니다. 

쓰레드 풀은 사용자의 연결이 많아져도 성능이 저하되는 것을 막기 위해 디자인되었습니다. MySQL 8.0 에서는 **"Max Transaction Limit"** 이 도입되어서, 동시에 실행할 수 있는 트랜잭션의 수에 제한을 두고 있습니다. 

## Max Transaction Limit

많은 요청이 동시다발적으로 들어오는 환경에서, MySQL 서버는 동시에 많은 트랜잭션을 다뤄야 합니다. 각 트랜잭션은 커밋 또는 롤백 될 때까지 테이블에 락을 겁니다. 시스템이 효과적으로 다룰 수 있는 수준 이상의 트랜잭션을 다루려고 하면, 시스템은 응답하는 것을 멈춘 것처럼 보일 수 있습니다. 따라서 서버의 TPS(throughput) 은 감소하게 됩니다.

따라서 Max Transaction Limit 은 동시에 실행할 수 있는 트랜잭션의 수에 제한을 둡니다. `thread_pool_max_transactions_limit` 시스템 변수에 0 이 아닌 값을 세팅하면 이 제한이 활성화됩니다. 0이 아닌 N은 쓰레드 풀은 동시에 N개 이하의 트랜잭션만 허용할 것이라는 것을 의미합니다. Max Transaction Limit 기능이 활성화되면, N은 런타임 내에 동적으로 변경할 수 있습니다. 

Max Transaction Limit 기능이 활성화되면, 각 쓰레드 그룹은 "thread_pool_max_transactions_limit" 을 쓰레드 그룹의 숫자로 나눈 개수 만큼의 트랜잭션을 동시에 실행할 수 있습니다. 명령어를 실행한 다음에는, 쿼리 워커 쓰레드는 같은 트랜잭션 내에서 다음 명령어를 고릅니다. 이 과정은 트랜잭션이 커밋되거나 롤백될때까지 진행됩니다. 

트랜잭션이 쓰레드 그룹에 동등하게 분배되기 위하여, Max Transaction Limit 은 쓰레드 풀 사이즈의 배수여야 합니다. 만약 배수가 아니라면, 가장 가까운 정수로 반올림될 것입니다. 

MySQL 에서는 트랜잭션의 수에만 제한을 두는 것이 아니라 `max_connections` 라는 시스템 변수를 사용해 최대 커넥션 수의 제한도 두고 있습니다. 

## 클라이언트에서 커넥션 풀 

본 포스팅을 정리한 이유는, 데이터베이스 클라이언트(어플리케이션 API) 에서 만드는 커넥션 풀과 MySQL 서버에서의 쓰레드 풀이 헷갈려서 입니다. 스택오버 플로우에는, 아래와 같이 나와있습니다. 

> Mysql thread pool works on the server. The connection pool is on the client (locally), it stores the connection so they don't have to be initiated again and again. The thread pool on mysql server, handles pool of workers that will actually communicate with clients to perform the sql operations.

클라이언트에서 다시 커넥션을 계속해서 만드는 비용을 줄이기 위해 만든것이 커넥션 풀이고, MySQL 서버 상에서 클라이언트 요청을 매번 새로운 쓰레드에 할당해서 처리하기 부담스러우니 만든 것이 쓰레드 풀이라고 이해하면 되겠습니다. 

### 클라이언트에서 커넥션 풀은 어떻게 구현되나요?

그렇다면 클라이언트 쪽에서 커넥션 풀을 만들었다면, 클라이언트는 쓰레드 풀은 안 만들어도 되는 것일까요? 웹 어플리케이션을 지탱하는 WAS 에서 DB 서버에 접근을 한다고 합시다. 이때 WAS 의 쓰레드 수와 커넥션 풀의 개수는 어떻게 설정해야 할까요? JDBC 드라이버를 사용하는 예시를 봅시다. 

```java
String driverPath = "net.sourceforge.jtds.jdbc.Driver";  
String address = "jdbc:jtds:sqlserver://IP/DB";  
String userName = "user";  
String password = "password";  
String query = "SELECT ... where id = ?";  
try {  
	 Class.forName(driverPath);  
	 Connection connection = DriverManager.getConnection(address, userName, password);  
	 PreparedStatement ps = con.prepareStatement(query);  
	 ps.setString(1, id);  
	 ResultSet rs = get.executeQuery();  
 // ....  
} catch (Exception e) { }  
} finally {  
	 rs.close();  
	 ps.close();  
}
```

1. DB 서버 접속을 위해 JDBC 드라이버를 로드한다.
2. DB 접속 정보와 DriverManager.getConnection() Method를 통해 DB Connection 객체를 얻는다.
3. Connection 객체로 부터 쿼리를 수행하기 위한 PreparedStatement 객체를 받는다.
4. executeQuery를 수행하여 그 결과로 ResultSet 객체를 받아서 데이터를 처리한다.
5. 처리가 완료되면 처리에 사용된 리소스들을 close하여 반환한다.

위에서 데이터베이스에서 원하는 데이터를 얻어오는 과정에서 가장 많은 시간이 소요되는 부분은 웹 서버에서 물리적으로 DB 서버에 최초로 연결해 Connection 객체를 생성하는 부분입니다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/eb0490a9-c464-469b-adfa-c16c2920c756)

이때 웹 서버 상에서 DBCP(데이터베이스 커넥션 풀)을 사용하면,  DB Connection 객체를 생성하는 부분에 대한 비용과 대기 시간을 줄이고, 네트워크 연결에 대한 부담을 줄일수 있습니다.

DBCP 는 HTTP 요청에 위의 1~5 까지의 단계를 거치지 않기 위한 방법입니다. 커넥션 풀을 이용하면 다수의 HTTP 요청에 대한 쓰레드를 효율적으로 처리할 수 있게 됩니다.

커넥션 풀의 구현체의 역할은 하기와 같습니다 :
1. WAS가 실행되면서 미리 일정량의 DB Connection 객체를 생성하고 `Pool` 이라는 공간에 저장해 둡니다.
2. HTTP 요청에 따라 필요할 때 Pool에서 Connection 객체를 가져다 쓰고 반환합니다.
3. 이와 같은 방식으로 HTTP 요청 마다 DB Driver를 로드하고 물리적인 연결에 의한 Connection 객체를 생성하는 비용이 줄어들게 됩니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a515bd5e-1408-416c-98fe-6457daf4650f)

아래의 값들은 커넥션 풀을 관리하기 위해 튜닝할 수 있는 값입니다. 

| Value       | Description                                        |
| ----------- | -------------------------------------------------- |
| maxActive   | 동시에 사용할 수 있는 최대 커넥션 개수                             |
| maxIdle     | Connection Pool에 반납할 때 최대로 유지될 수 있는 커넥션 개수         |
| minIdle     | 최소한으로 유지할 커넥션 개수                                   |
| initialSize | 최초로 getConnection() Method를 통해 커넥션 풀에 채워 넣을 커넥션 개수 |

아래의 그림을 살펴보면 Pool에 Connection 을 Idle(대기) 상태로 두었다고, 실제 사용하는(Active) 모습입니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/5d054c74-f2b7-4d2e-bb19-b92a59d1cd48)

- maxActive >= initialSize
- maxActive = maxIdle


### WAS의 쓰레드 풀수 커넥션 풀의 관계

WAS 에서 성능에 가장 많은 영향을 주는 부분이 **쓰레드와 커넥션 풀의 개수**입니다. 이들 값은 직접적으로 메모리와 관련이 있기 때문에, 많이 사용하면 할 수록 메모리를 많이 점유하게 됩니다. 그렇다고 반대로 메모리를 위해 적게 지정한다면, 서버에서는 많은 요청을 처리하지 못하고 대기 할 수 밖에 없습니다.

WAS 에서 **쓰레드의 수는 커넥션 풀의 개수보다 여유있게 설정**하는 것이 바람직합니다. 왜냐하면 어플리케이션에 대한 모든 요청이 DB 접근에 접근하는 것이 아니기 때문입니다. 또한 동시성 관점에서, DB 요청을 담당한는 스레드 풀을 따로 가지고 있으면 좋습니다. 

예를 들어 스프링 MVC 로 개발시, WAS 로 톰캣을 띄운다고 해봅시다. 이때 톰캣은 브라우저나 프런트 서버에서 들어온 요청을 처리하는 리퀘스트 처리 스레드 풀을 기본적으로 갖고 있습니다. 그리고 톰캣은 MySQL 같은 데이터베이스에 요청을 할 때 사용되는 스레드 풀도 가지고 있어야 합니다. **왜 따로 쓰레드 풀을 설정해야 할까요?** 

그 이유는, 스프링 MVC는 자바에서 제공하는 서블릿 기반인데, 서블릿 컨테이너는 요청이 들어왔을 때 해당 요청 당 스레드 하나를 매핑해줍니다. 따라서 전체 요청과 응답까지 그 하나의 스레드가 처리를 해주는 형태입니다. 만약 스프링 개발을 하면서 비동기 관련 설정을 하지 않았다면, 모든 요청은 톰캣의 기본 스레드풀 에서만 작동합니다. 이렇게 되면 클라이언트의 요청을 처리하는 과정에서 DB의 데이터를 읽거나 쓰는는 과정이 발생하면, 톰캣의 쓰레드를 블로킹을 하고 DB의 응답이 올때까지 계속 기다려야 합니다. DB에 오래 걸리는 작업들이 많이 수행되는 서버라면, 모든 클라이언트 요청을 처리하던 쓰레드가 DB에 블로킹이 걸려 모두 블로킹 상태에 빠질 수 있습니다. 

따라서 블로킹 I/O 와 연결된 부분들에 대해 별도의 스레드 풀을 만들어 할당을 해주면, 톰캣에서 시작된 스레드가 DB 요청 용도로 만들어 놓은 스레드로 갈아타고, 실제로 DB 의 응답을 기다리는 것은 DB 용도로 만들어 놓은 스레드가 기다리게 됩니다. 그렇다면 톰캣의 스레드는 반납이 되어, 다른 클라이언트의 요청을 처리할 수 있습니다. 

음.. 여기서 의문이 생길 수 도 있습니다. **"그렇다면 쓰레드 풀을 따로 만드는 것이 아니라 쓰레드의 개수를 무진장 늘리면 되는 것이 아닌가요?"** 라고 생각할 수 도 있습니다. 하지만 쓰레드 풀이 1개라는 의미는, 아무리 쓰레드가 많아도 그 모든 쓰레드가 블로킹 I/O 에 빠지는 순간 더 이상 할당할 수 있는 쓰레드가 없습니다. 그렇지만 쓰레드 풀이 분리가 되면, 쓰레드 풀이 갖고 있는 쓰레드 개수보다 훨씬 많은 개수를 태스크 큐에 쌓아놓을 수 있습니다. 그러면 DB 요청들은 따로 DB 스레드 풀의 태스크 큐에 넣어두고, 톰캣의 스레드는 CPU 와 관련된 연산들을 계속 할 수 있습니다. 

다른 이유로는 장애와 관련된 이유도 있습니다. 만약 쓰레드 풀을 1개만 사용하는데, DB에 장애가 발생했다면 쓰레드 풀 내 모든 쓰레드가 DB 요청을 하게 되고 쓰레드 풀 전체가 사용중인 상태가 됩니다. 하지만 쓰레드 풀을 분리하면, DB를 사용하지 않는 쓰레드 풀은 DB의 장애에 영향을 받지 않을 수 있습니다. 따라서 장애 상황에서 환경 분리도 할 수 있습니다. 

최적의 성능을 위해서는 실제로 요청이 얼마 들어오는지 파악한 후, 성능 테스트를 통해 쓰레드의 수와 커넥션 풀의 수, 쓰레드 풀을 적절하게 설정하는 것이 바람직합니다.  


## Reference
- [The New MySQL Thread Pool](https://dev.mysql.com/blog-archive/the-new-mysql-thread-pool/)
- https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_max_connections
- [7.1.12.1 Connection Interfaces](https://dev.mysql.com/doc/refman/8.0/en/connection-interfaces.html)
- https://stackoverflow.com/questions/28769963/jdbc-connection-pool-and-mysql-thread-pool
- https://d2.naver.com/helloworld/5102792
- https://www.holaxprogramming.com/2013/01/10/devops-how-to-manage-dbcp/