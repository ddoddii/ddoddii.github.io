+++
author = "Soeun"
title = "[데이터베이스] Transaction, Concurrency Control, Recovery, Locking"
date = "2023-11-20"
description = "트랜잭션의 의미와 동시성 제어를 하는 이유, 2PL 까지"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
math = true
slug = "transaction"
+++

## Intro to Transaction

### Single vs. Multi user

Transaction 을 더 깊이 다루기 전에, 우선 사용자들이 DB 를 사용하는 상황에 대해 생각해보자. 이것은 동시에(concurrently) 사용하는 사용자가 하나(single-user) 혹은 다수(multi-user) 이냐에 따라 나눌 수 있다. 

다수의 사용자는 multiprogramming 이라는 컨셉 덕분에 DB 를 동시에 사용할 수 있다. 왜냐면 OS에서 다수의 프로그램 또는 프로세스를 동시에 실행시킬 수 있기 때문이다. 

아래는 single CPU vs. multiple CPU 를 비교한 것이다. single CPU 는 최대 1개의 프로세스만 실행할 수 있다. 따라서 A,B는 번갈아가며 (interleaving) 실행된다. 그럼에도 multiprogramming 덕분에 A,B 가 동시에 실행되는 것처럼 보이는데, multiprogramming OS 가 하나의 프로세스에서 명령어들을 실행시키고 중지한 다음, 다른 프로세스의 명령어들을 실행한다. 

반면 multiple CPU 면 parallel processing 을 할 수 있다. C,D 처럼 2개의 프로세스를 동시에 실행시킬 수 있다. 

<img width="520" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b81d4daa-5bc2-45c8-bf58-bae22dcf38ae">

하지만 여기서는 interleaved 된 경우에 동시성 조절을 어떻게 할 것인지 살펴볼 것이다. 

### More about Transactions

> Transaction is a logical unit of database processing that must be completed in its entirety to ensure correctness. 

트랜잭션 은 DB의 컨텐츠를 접근하거나, 바꾸는 프로그램의 명령이다. 트랜잭션은 하나 이상의 DB 접근 명령을 수행한다 - insertion ,deletion, modification, retrieval. 트랜잭션을 알리는 방법에는 명령어 전에 'begin transaction' 을 표기하고, 명령어 후에 'end transaction' 을 표기하는 것이 있다. 만약 트랜잭션이 DB 를 업데이트 하지 않고 오로지 정보를 받아오기만 하면, **read-only transaction** 이라고 하고, 그게 아니면 **read-write transaction** 이라고 한다. 

트랜잭션의 예시를 몇개 보자.
(a) 
```sql
exec sql update ACCOUNTS set account_total =account_total*(1+interest_rate);
```

(b)
```sql
done := 0;
while (done < max_account) {
	exec sql begin transaction; 
	exec sql update ACCOUNTS
		set account_total =account_total*(1+interest_rate);
		where account_no between done +1 to done + :stepsize; 
	exec sql update BATCH
		set done = done + :stepsize; 
	exec sql commit;
	done = done+ stepsize;
}
```

(a) 는 모든 계좌(5000만개)에 대해 동시에 업데이트를 수행한다. SQL 는 하나하나가 트랜잭션이다. 즉, 다 수행되거나 다 수행되지 않거나 이다. SQL 라인이 1개라면 전체 계산이 끝나야 트랜잭션이 끝난다. 모든 과정을 하나로 때려박은 것을 **full-batch** 라고 한다. 

(b) 는 계산 과정을 batch 여러 개로 나누었다. 여기서는 배치 개수를 나누어서 (1000만개) 업데이트를 수행한다. BATCH 에는 done 이라는 속성 1개만 있다. 서버가 SQL 수행 과정에 죽으면, 메모리가 날라가서 지금까지 계산한 계좌 수를 날릴 수 도 있다. 하지만 서버가 죽어도 디스크는 살아남으니 지금까지 계산한 변수를 BATCH 에 저장해 놓는다. 이것을 **mini-batch** 라고 한다. 

`sql commit` 하기 전에는, 모든 연산 결과는 메모리 영역에 있다. 


### Properties of Transaction(ACID)

트랜잭션이 성공적으로 처리되어 DB의 무결성이 보장되려면, 4가지 특성을 만족해야 한다. 이것을 ACID 라고 한다. 

#### Atomicity(원자성)
원자성은 트랜잭션이 atomic unit of processing 이라는 것이다. 이것은, 트랜잭션을 구성하는 연산들이 모두 정상적으로 실행되거나 하나도 실행되지 않아야 한다는 all-or-nothing 방식을 의미한다. 

원자성을 보장하려면 recovery 기능이 필수적이다. 

#### Consistency perservation(일관성)
트랜잭션이 성공적으로 수행된 후에도 DB 가 일관된 상태를 유지해야 한다. 즉, 트랜잭션이 수행되기 전 일관된 상태였다면 트랜잭션을 수행한 후에도 또 다른 일관된 상태여야 한다. 

#### Isolation(격리성)
현재 수행 중인 트랜잭션이 완료될 때까지 트랜잭션이 생성한 중간 연산 결과에 다른 트랜잭션이 접근할 수 없다. 이것은 동시성 제어가 보장한다. 

#### Durability(지속성)
지속성은 트랜잭션이 성공적으로 완료된 이후 DB에 반영한 수행 결과는 어떠한 경우에도 손실되지 않고 영구적이어야 한다. 

DBMS 는 트랜잭션의 4가지 특성을 보장하기 위해 기능을 제공한다. 

| 트랜잭션의 특성 | DBMS 기능           |
| --------------- | ------------------- |
| 원자성          | recovery            |
| 일관성          | concurrency control |
| 격리성          | concurrency control |
| 지속성          | recovery            | 


### Transaction Operations

트랜잭션 연산에는 commit 과, rollback 이 있다. 
- commit : 트랜잭션이 성공적으로 수행되었음을 알림
- rollback : 트랜잭션을 수행하는데 실패했음을 선언

commit 이 실행되면, 트랜잭션 수행 결과가 DB 에 반영된다. rollback 이 실행되면, 트랜잭션이 지금까지 실행한 연산의 결과가 취소되고 트랜잭션이 수행되기 전의 상태로 돌아간다. 

### Transaction status

트랜잭션의 상태는 5가지가 있다. 

<img width="400" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/75b5115c-f006-40d6-9a2e-271867b4508f">

- 활동(active)
  
  트랜잭션이 수행되기 시작하여 현재 수행 중인 상태이다 . 

- 부분 완료(partially committed)
  
  트랜잭션의 마지막 연산이 실행된 직후를 부분 완료 상태라 한다. 이것은 트랜잭션의 모든 연산을 처리한 상태이다. 하지만 아직 결과를 DB 에 최종 반영하지는 않은 상태이다. 

- 완료(committed)
  
  트랜잭션이 성공적으로 완료된 상태이다. 이때 트랜잭션이 수행한 최종 결과를 DB에 반영하고, DB 가 새로운 일관된 상태가 되며 트랜잭션이 종료된다. 

- 철회(aborted)
  
  트랜잭션을 수행하는데 실패하여 rollback 연산을 실행한 상태를 철회 상태라고 한다. 이때는 지금까지 실행한 모든 트랜잭션 연산을 취소하고, 트랜잭션이 수행되기 전의 DB 상태로 되돌린다. 철회 상태로 종료된 트랜잭션은 상황에 따라 다시 수행되거나 폐기된다. 하드웨어 / 소프트웨어의 오류로 트랜잭션 수행이 중단되면 다시 시작한다. 하지만 트랜잭션의 논리적인 오류라면 폐기한다. 

## Recovery

트랜잭션의 특성을 보장하기 위해 recovery 시스템은 필수적이다. 시스템이 제대로 동작하지 않는 것을 failure 라고 하는데, 원인은 다양하다. 

DB 는 기본적으로 저장 장치에 저장된다. 저장 장치는 장애가 발생했을 때 대응 방법이 모두 다르다.

| 저장 장치                      | 설명                                                                                                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 휘발성(volatile) 저장 장치     | 장애가 발생하면 데이터가 손실됨. ex) 메인 메모리                                                                                                                          |
| 비휘발성(involatile) 저장 장치 | 장애가 발생해도 저장된 데이터가 손실되지 않음. 하지만 디스크 헤더 손상같은 저장 장치 자체에 이상이 발생하면 데이터가 손실될 수 있다. <br> ex. 디스크, 자기 테이프, CV/DVD |
| 안정(stable) 저장 장치         | 비휘발성(involatile) 저장 장치를 이용해 복사본 여러 개를 만드는 방법으로, 장애가 발생해도 데이터가 손실되지 않고 영구적을 저장됨                                          | 


DB는 named data items 의 집합이다. data item 의 사이즈를 granularity 라고 한다. data item 은 database record 가 될 수 도 있지만, disk block 과 같이 더 큰 유닛이 될 수 도 있고, 훨씬 작은 개별의 attribute value 가 될 수 도 있다. 각 data item 은 개별적으로 이름이 있다. 만약 data item granularity 가 하나의 disk block 이면, disk block address 가 data item 이름이 될 수 있다. 
이 정의에 따라서 데이터베이스 트랜잭션을 다시 보자. 

- read_item(X) : DB item X 를  프로그램 변수로 읽어온다.
- write_item(X) : 프로그램 변수 X 를 DB item X 로 저장한다. 

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1a22953a-a0a9-4e4c-848e-1e98ea84fa12">


디스크와 메인 메모리 사이에 가장 기본적인 데이터 전송 단위는 disk page(disk block) 이다. **read_item(X)** 는 다음과 같은 순서로 실행된다. 
1. item X 를 포함하는 disk block 의 주소를 찾는다.
2. 그 disk block을 메인 메모리의 버퍼로 복사한다. 
3. 버퍼에서 item X 를 꺼내와서 프로그램 변수 X에 복사한다. 


**write_item(X)** 는 다음의 순서로 실행된다. 
1. item X 를 포함하는 disk block 의 주소를 찾는다.
2. 그 disk block을 메인 메모리의 버퍼로 복사한다. 
3. 프로그램 변수 X를 버퍼의 알맞은 장소에 있는  item X 에 복사한다.
4. 버퍼로부터 업데이트 된 disk block 를 디스크에 다시 저장한다. (바로 혹은 조금 이따가)

Step 4 에서 실제로 데이터베이스의 disk 에 업데이트가 일어난다. 변경된 disk block 을 실제로 저장하는 시점은 DBMS 의 recovery manager 에 의해 결정된다. DBMS 는 database cache 에 메인 메모리에 있는 data buffer 의 수를 저장한다. 각 버퍼는 대체로 하나의 disk block 의 컨텐츠를 저장하고 있다. 만약 버퍼가 모두 사용중이고 추가적으로 disk block 이 메모리에 복사되어야 하면, 대체될 버퍼를 고르는 buffer replacement policy 가 필요하다. 가장 흔한 것은 LRU(Least Recently Used) 이다. 

다수의 사용자들이 read_set, write-set 를 동시에 수행할 수 있다. 이때 같은 item 에 대해 update 가 수행될 수 있다.  이것은 데이베이스 불일치로 이어질 수 있다. 그러므로 동시성 조절(concurrency control) 는 필수적이다. 또, 만약 서버가 셧다운 되어서 트랜잭션이 날라가면 recovery 도 필수적이다. 

### Recovery technique

recovery 는 DBMS 의 recovery manager가 담당한다. DB 회복의 핵심은 데이터 중복이다. 데이터를 별도의 장소에 미리 복사해놓았다가, 장애가 발생했을 때 복사본을 이용해서 원상복구한다. 

- **dump** : DB 전체를 다른 저장 장치에 주기적으로 복사하는 방법
- **log** : DB 안에서 변경 연산이 실행될 때마다 데이터를 변경하기 이전 값과 이후의 값을 별도의 파일에 저장해 놓는 것 

장애가 발생했을 때 데이터를 복구하는 가장 기본적인 방법은 redo 나 undo 를 실행하는 것이다. 
- **redo** : 가장 최근에 변경한 DB 복사본을 가져 온 후 로그를 이용해 복사본이 만들어진 이후에 실행된 모든 연산을 재실행 
- **undo** : 로그를 이용해 지금까지 실행된 모든 변경 연산을 취소함 

DB 복구에는 로그 파일이 중요한 역할을 한다. 로그 파일은 레코드 단위로 기록된다. 로크 파일의 레코드는 4가지 유형이 있다. 

| 로그 레코드 종류               | 설명                                                                      |
| ------------------------------ | ------------------------------------------------------------------------- |
| <$T_i$ , start>                | 트랜잭션 $T_i$ 가 수행을 시작했음을 기록                                  |
| <$T_i$ ,X,old_value,new_value> | 트랜잭션 $T_i$ 가 old_value -> new_value 로 변경한 연산을 실행했음을 기록 |
| <$T_i$ , commit>               | 트랜잭션 $T_i$ 가 성공적으로 완료되었음을 기록. commit point 라고 한다.                            |
| <$T_i$ , abort>                | 트랜잭션 $T_i$ 가 철회 되었음을 기록                                      | 

**commit point** 는 모든 트랜잭션이 성공적으로 완료되었고, 로그 파일에도 기록된 시점이다. 

#### log recovery method

로그 회복 기법은 데이터를 변경한 연산 결과를 DB 에 반영하는 시점에 따라 immediate update recovery / deferred update recovery 로 나뉜다. 

**immediate update** 는 트랜잭션 수행 중에 데이터를 변경한 연산 결과를 즉시 DB 에 반영한다. 그리고 이 내용을 로그 파일에도 기록한다. 장애를 회복하려면, 로그 파일을 먼저 기록하고 DB 변경 연산을 수행해야 한다. immediate update 에서는 장애가 발생하면 redo 나 undo 연산을 실행한다. 어느 연산을 할지는 2가지 경우로 나누어진다. 
- 트랜잭션이 완료되기 전에 장애 발생 (<$T_i$ , start> 는 존재, <$T_i$ , commit>  는 존재X ) -> undo 
- 트랜잭션이 완료된 후에 장애 발생(<$T_i$ , start> , <$T_i$ , commit>  모두 존재) -> redo 

**deferred update** 는 트랜잭션이 수행되는 동안에는 데이터 변경 연산의 결과가 DB 에 바로 반영되지 않고 로그 파일에만 기록해 두었다가, 트랜잭션이 부분 완료된 후에 로그에 기록된 내용을 이용해 DB 에 한번에 반영한다. 즉 commit point 이후에 DB 에 실제 변경된 결과가 반영된다. 

만약 트랜잭션 중간에 장애가 발생하면, 로그에 기록된 파일을 버리기만 하면 되므로 undo 연산은 필요없다. redo 연산만 필요하므로 로그 레코드에 변경 이전 값을 기록할 필요가 없다. 따라서 변경 연산 실행에 대한 로그 레코드는 <$T_i$, X, new_value> 형식으로 기록된다. 

이때도 장애가 발생했을 때 deferred update recovery 가 취하는 조치들을 보자. 
- 트랜잭션이 완료되기 전에 장애 발생 -> 로그 내용을 무시하고 버림 
- 트랜잭션이 완료된 후에 장애 발생 -> redo 

#### checkpoint recovery method

로그를 이용한 회복 기법은 모든 로그를 분석해야 하므로 시간이 너무 오래 걸린다. 
그리고 system crash 가 되었을 때, 이전 트랜잭션이 commit 된 이후라도 deferred update 를 사용했으면 실제로 write 한 시점이 언제가 될 지 모른다. 따라서 모든 deferred 트랜잭션을 전부 살펴봐야 한다. 이 때 무한대로 과거로 가는 것이 싫어서 도입된 것이 **checkpoint** 이다. 

checkpoint recovery method 는 로그 회복 기법과 같은 방법으로 로그 기록을 이용하되, 주기적으로 checkpoint 를 만들어둔다. checkpoint 에서는 이전에 커밋된 모든 트랜잭션을 write 하는 작업을 수행한다. 

<img width="600" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/1c4a2ec1-3807-44a0-b6ba-7a68a423af5c">


## Concurrency Control
### Why Concurrency Control is needed

비행기 좌석 예약 시스템의 예시를 보자. 2개의 트랜잭션이 있다. 
- T1 : cancels N reservations in X flight & reserves N seats in Y flight
- T2 : reserves M seats in X flight

<img width="523" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/8f29ee8c-fb60-4271-8c92-3cffc84a727e">

이때, 아래와 같은 문제들이 생길 수 있다. 
#### Lost update problem
같은 DB item 에 1개 이상의 트랜잭션이 access & update 할 때 발생한다. 
초기에, X= 80, N = 5, M = 4 라고 하자.

![IMG_8F480EE28E75-1](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/927d3df4-dce5-4aa1-8e69-96f687581b12)

처음에, T1 에서 X:= X-N 이 실행되면 X = 75이다. 이것은 아직 메모리 영역에 있으므로 DB 의 디스크에는 영향이 없다. 따라서 T2 에서 X:=X+M 이 실행될때는 X 의 초기값은 여전히 80 이고, 실행후에는 X = 84 이다. T1 에서 write 를 하면 실제 디스크에 X = 75 가 저장된다. 하지만 T2 에서 write 를 하면 X 의 값은 84 업데이트 된다. 따라서 T1 의 값은 overwritten 되어, lost 된다. 

이 문제는 X 에 lock 를 걸지 않아서 생기는 문제이다. 따라서 동시성 조절이 필요하다. 

#### Temporary update problem

만약에 하나의 트랜잭션이 DB item 을 업데이트하고, 망가졌다고 하자. 업데이트 된 아이템은 원래 값으로 돌려놓기 이전에 다른 트랜잭션에 의해 사용된다. 

 ![IMG_F3224B02DA7E-1](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/f5cb83f7-f26c-4087-8058-c217588d62b5)

여기서는 T1 에서 read 하고 write 까지 하므로 업데이트 된 x = 75 값이 디스크에 저장된다. 그러면 T2 에서 이것을 읽어와서 연산을 수행 후 다시 디스크에 업데이트된 x 값, x = 79 를 저장한다. 여기서 T1 의 read_item(Y) 에서 오류가 발생했다고 하자. 그럼 rollback 을 해서 T1에서 했던 연산을 취소해야 한다. X 값에 N을 다시 더하면 X = 79 +5 = 84 가 된다. 따라서 값의 불일치가 생긴다. 


## Locking
locking 은 병행 수행되는 트랜잭션들이 동일한 데이터에 동시에 접근하지 못하도록 lock  과 unlock 라는 2개의 연산을 이용하여 제어한다. Locking 의 기본 원리는 한 트랜잭션이 먼저 접근한 데이터에 대한 연산을 모두 마칠 때까지, 해당 데이터에 대해 다른 트랜잭션이 접근하지 못하도록 상호 배재(mutual exclusion) 하여 직렬 가능성을 보장하는 것이다. 

트랜잭션이 Locking을 사용하려면, 데이터에 read / write 연산을 실행하기 전에 반드시 lock 연산을 실행해야 한다. 다른 트랜잭션이 이미 lock 연산을 실행한 데이터에 대해서는 다시 lock 연산을 수행할 수 없다. 모든 연산이 수행된 후에는 unlock 을 이용해 독점권을 반납해야 한다. 

어느 연산인지에 따라 제약을 달리할 수 도 있다. write 는 엄격하게 제어해야 하지만, read 는 다수가 동시에 해도 문제 없다. 따라서 lock 을 두 종류로 구분할 수 있다. 

| 연산           | 설명                                                                                                                                               |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| shared lock    | shared lock 연산을 수행하면, read 는 할 수 있지만 write 는 하지 못한다.<br>  해당 데이터에 대해 다른 트랜잭션도 shared lock 연산을 수행할 수 있다. |
| exclusive lock | 해당 데이터에 대해 read, write 모두 수행할 수 있다.<br>  다른 트랜잭션은 어떠한 lock 연산도 수행하지 못한다.                                            | 

Lock 끼리 연산의 양립성을 보자.

|                | shared lock | exclusive lock |
| -------------- | ----------- | -------------- |
| shared lock    | O           | X              |
| exclusive lock | X           | X               |

하지만, locking 만으로는 모든 문제를 해결할 수 없다. 

<img width="492" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/40cf0643-b331-4a6f-a2bb-bbd8eab0223b">

위의 경우에, T1, T2 모두 locking 연산을 수행했지만 잘못된 연산 결과가 나온다. 이유는 T1이 너무 빨리 unlock 을 실행하여 T2 가 일관성 없는 데이터에 접근했기 때문이다. 이 때문에 Two Phase Locking 이 나왔다. 


### Two Phase Locking(2PL)

2PL 을 따르려면 모든 트랜잭션이 lock, unlock 을 다음과 같이 2단계로 나누어 실행해야 한다. 
- **expanding stage** : 트랜잭션이 lock 만 실행할 수 있고, unlock 는 실행할 수 없다.
- **shrinking stage** : 트랜잭션이 unlock 만 실행할 수 있고, lock 는 실행할 수 없다.

<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b21d901c-6b23-4941-bc00-abb254ae91fa">

2PL 을 적용하면, 트랜잭션 스케쥴의 직렬 가능성을 보장할 수 있다. 하지만 **deadlock** 상황이 발생할 수 도 있다. deadlock 은 트랜잭션들이 상대가 독점하고 있는 데이터에 unlock 연산이 실행되기를 서로 기다리면서 수행을 중단하고 있는 상태이다. 이 교착상태에 빠지면 트랜잭션들은 더 이상 수행되지 못하고 상대 트랜잭션이 먼저 unlock 연산을 해주길 기다린다. 


## Reference
- Fundamentals of database systems, 7th edition, ch.20
- 2023-2 데이터베이스, 이원석 교수님
- 데이터베이스 개론