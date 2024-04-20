+++
author = "Soeun"
title = "[데이터베이스] RDB에서 Key"
date = "2023-09-19"
summary = "Key의 종류와 개념"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
series = ["Database"]
series_order = 1
slug = "key"
+++

## RDB 의 구성

RDB 에서 테이블은 스키마와 여러개의 인스턴스들로 구성되어 있다. 

|고객아이디|고객이름|나이|전화번호|
|----|---|---|---|
|apple|소은|25|01012345678|
|eagle|연세|80|0221231234|
|pudding|곰돌|30|Null|

스키마는 테이블 이름과 테이블에 포함된 모든 속성의 이름을 정의하는 테이블의 논리적 구조이다. 위의 테이블에서 고객아이디, 고객이름, 나이, 전화번호가 스키마에 해당한다. 

인스턴스는 한 시점에 테이블에 존재하는 투플들의 집합이다. 인스턴스에 포함된 투플들을 테이블에서 정의하는 각 속성에 대응하는 실제 값으로 구성되어 있다. 위의 테이블에는 3개의 투플이 존재한다. 

스키마는 자주 변하지 않지만, 인스턴스는 투플의 삽입, 삭제, 수정이 자주 발생한다. 

일반적으로 데이터베이스는 테이블 여러개로 구성되어 있다. 데이터베이스 스키마는 데이터베이스를 구성하는 테이블들의 스키마를 모아놓은 것이다. 

## 테이블의 특징
RDB 의 테이블에는 4가지 특징이 있다. 
1. 투플의 유일성 : 하나의 테이블에는 동일한 투플이 존재할 수 없다.
2. 투플의 무순서 : 하나의 테이블에서 투플 사이의 순서는 무의미하다. 
3. 속성의 무순서 : 하나의 테이블에서 속성 사이의 순서는 무의미하다. 
4. 속성의 원자성 : 속성 값으로 원자 값만 사용할 수 있다. 

## Key 의 종류 ⭐️

각 투플을 구별하려면 어떻게 해야 할까? 모든 속성을 이용하는 것보다 일부 속성만 이용해서 구분하는 것이 효과적일 수 있다. 따라서 테이블에 포함된 투플들을 유일하게 구별해주는 역할은 속성 또는 속성들의 집합인 Key 가 담당한다. 

Key 에는 5가지 종류가 있다 : Primary Key, Foreign Key, Alternate Key, Super Key, Candidate Key

### Super Key
> Super Key 는 유일성의 특성을 만족하는 속성 또는 속성들의 집합이다. 

유일성은 키가 갖추어야 하는 기본 특성으로, 하나의 테이블에서 키로 지정된 속성 값은 투플마다 달라야 한다는 의미이다. 위의 테이블에서 고객아이디는 고객마다 다르므로 슈퍼키가 될 수 있다. 하지만 나이가 같은 고객이 존재할 수 있으므로 나이는 슈퍼키가 되지 못한다.  (고객아이디, 고객이름) 으로 구성된 속성 집합도 슈퍼키가 될 수 있다. 

### Candidate Key 
> Candidate Key (후보키) 는 유일성과 최소성을 만족하는 속성 또는 속성들의 집합이다. 

최소성은 꼭 필요한 최소한의 속성들로만 키를 구성하는 특징이다. 슈퍼키 중에서 최소성을 만족하는 것이 후보키가 된다. 고객아이디만으로 고객 투플을 구별할 수 있으므로 (고객아이디, 고객이름) 은 최소성을 만족하지 못하므로 후보키가 되지 못한다. 

### Primary Key
> Primary Key는 여러 후보키 중에서 기본적으로 사용할 키이다. 

위의 테이블에 고객 주소가 포함된다고 하자. 그렇다면 (고객이름, 고객주소) 도 후보키가 될 수 있다. 그러면 고객아이디 vs. (고객이름, 고객주소) 중에 무엇을 기본키로 할지 선택해야 한다. PK 를 선택할 때 고려할 사항에 대해 알아보자. 

- Null 값을 가질 수 있는 속성이 포함된 후보키는 PK 로 부적합하다. 
- 값이 자주 변경될 수 있는 속성이 포함된 PK 는 부적합하다. 
- 단순한 후보키를 PK 로 설정한다. 

PK 는 Null 값을 가지면 안되고, 유일성과 최소성을 모두 만족해야 한다 ! 각 테이블 마다 PK 는 오직 1개이다. 오해하면 안되는게, 그렇다고 속성 단 하나만 PK 여야 되는 것은 아니다. 정말 단 하나의 속성으로 투플이 구분되지 않는다면 두 가지 속성의 집합이 PK 가 될 수도 있다. 그리고 꼭 auto-incrementing id 가 PK 가 될 필요는 없다. 그저 convention 인 것이지, 필수적인 규칙은 아니다. 

### Alternate Key
> Alternate Key 는 Primary Key 로 선택되지 못한 후보키들이다. 

<img width="448" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/41f32b14-99fd-4f36-9a1d-790df3fe11c1">

### Foreign Key 
> Foreign Key 는 어떤 테이블에 소속된 속성 또는 속성 집합이 다른 테이블의 기본키가 되는 키이다. 

고객 테이블 : 고객 (**고객아이디**, 고객이름, 나이, 등급, 직업)

주문 테이블 : 주문 (주문번호, **주문고객**, 주문제품, 수량, 단가, 주문날짜)

고객 테이블에서 고객아이디는 PK 이다. 주문 테이블에서 주문고객 속성이 고객 테이블의 PK 인 고객아이디 속성을 참조하면 주문고객 속성이 FK 가 된다. FK는 반드시 다른 테이블의 PK 를 참조해야 한다. 

FK는 반드시 다른 테이블의 PK 를 참조할 필요는 없다. 자기 자신의 PK 를 참조해도 된다. 만약 고객 테이블에 추천 고객 속성이 있다면, 이것은 고객테이블의 고객아이디 속성을 참조해도 된다. 

FK는 PK를 참조하지만, PK 가 아니기 때문에 Null 값을 가질 수 있다. 추천 고객이 아무도 없으면 Null 값을 넣을 수 있다. 

-----

## 더 알아보기

### 기본키는 수정이 가능한가요?

기본키가 다른 테이블의 외래키에 의해 참조되는 경우와 그렇지 않은 경우로 나뉜다. 

1. **기본키가 다른 테이블의 외래키에 의해 참조되는 경우 (= 연관 관계가 있는 경우)**
	
	만약  board table의 board_id 가 FK로 user table 의 user_id 를 참조한다고 하자. 연관 관계에 있는 부모 행은 변경이나 삭제가 불가능하다. 
		<img width="564" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/75d24263-4d85-4de3-96e0-90bc0d11675f">
	
	실제로 업데이트 해보면 아래와 같은 error 가 뜬다. 

	```SQL
	update users set user_id = 3 where name = 'user2';
	ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key 
	constraint fails (`test`.`board`, CONSTRAINT `ex_fk` 
    FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`))
	```

	**그러면 외래키의 경우에는 ?** 
	
	만약 변경하고자 하는 외래키가 user_id 에 없는 id 라면 변경이 불가능하다. 
	
	<img width="521" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cb6428ef-a5df-460d-97d4-a1164bde82a2">
	그러나 변경하고자 하는 외래키가 user_id (부모) 에 있는 id 값이라면 변경 가능하다. 
	
	<img width="527" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/12259ece-fe9c-45f1-b2b7-d4d4ffa1dabd">

2. **연관 관계가 없는 경우**
	
	기본키는 연간 관계가 없다면 변경이 가능하다.
	
	<img width="260" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bb6a43e6-23c5-4c8a-a85c-26e49bdbfcc2">


### 사실 MySQL의 경우, 기본키를 설정하지 않아도 테이블이 만들어집니다. 어떻게 이게 가능한 걸까요?

MySQL documentation 에서 기본키를 지정하면 MySQL의 InnoDB 가 어떻게 동작한는지 보자. 

> When you define a `PRIMARY KEY` on a table, `InnoDB` uses it as the clustered index. A primary key should be defined for each table. If there is no logical unique and non-null column or set of columns to use a the primary key, add an auto-increment column. Auto-increment column values are unique and are added automatically as new rows are inserted.

InnoDB 가 사용자가 지정한 기본지를 clustered index 로 사용한다.  기본키를 유일하고, non-null 속성 또는 속성들의 집합으로 설정거나, 이 조건을 만족하는 속성이 없다면 auto-increment column 을 추가해서 기본키로 설정하라고 한다. 

그렇다면  InnoDB 가 기본키가 없이 어떻게 동작하는지 보자. 

> If you do not define a `PRIMARY KEY` for a table, `InnoDB` uses the first `UNIQUE` index with all key columns defined as `NOT NULL` as the clustered index.
  
해석해보자면, InnoDB (MySQL의 데이터베이스 엔진) 이 Null 값이 없는 첫번째 Unique Index 를 clustered index 로 사용한다. 하지만 이 clustered index 는 데이터베이스 스키마에서 볼 수 없다. 그러니까 사실상 InnoDB 가 모든 속성들을 다 훑어보면서 기본키로 쓸만한 속성이 있나 찾아보는 거다. 

그렇지만 만약 이 조건을 만족하는 속성이 없다면 ? 

> If a table has no `PRIMARY KEY` or suitable `UNIQUE` index, `InnoDB` generates a hidden clustered index named `GEN_CLUST_INDEX` on a synthetic column that contains row ID values. The rows are ordered by the row ID that `InnoDB` assigns. The row ID is a 6-byte field that increases monotonically as new rows are inserted. Thus, the rows ordered by the row ID are physically in order of insertion.

 InnoDB 가 `GEN_CLUST_INDEX` 라는 clustered index 를 직접 만든다. 
이 clustered index 에 대해서는 https://mangkyu.tistory.com/285 에 자세히 설명되어 있다. 
### 외래키 값은 NULL이 들어올 수 있나요?

외래키 값은 NULL 을 허용한다. 그리고 중복도 허용한다. 외래키는 단순히 부모테이블의 기본키를 참조하는 값이기 때문이다 !

외래키의 제약조건은 값이 들어올 때는 부모 테이블(참조되는 테이블)의 기본키에 존재하는 값만 들어올 수 있다는 것이다. 

### 어떤 칼럼의 정의에 UNIQUE 키워드가 붙는다고 가정해 봅시다. 이 칼럼을 활용한 쿼리의 성능은 그렇지 않은 것과 비교해서 어떻게 다를까요?

- 쓰기 관점
	
	만약 Column 에 UNIQUE 키워드가 붙으면, 쓸 때 정말 느리다. 왜냐하면 데이터를 넣을 때 트리를 타고 들어가면서 매번 중복되는지 확인해주어야 한다. 
- 읽기 관점
	
	UNIQUE 제약 조건이 있는 칼럼을 검색 조건으로 사용할 때, 데이터베이스는 해당 칼럼에 대한 인덱스를 생성한다. 이 인덱스는 데이터 검색을 빠르게 해주므로 쿼리의 성능이 향상된다. 

결국 쓰기 능력을 잃고 읽기 능력을 얻는 것이 아닌가 싶다 .. 
## Reference
- 데이터베이스 개론 3판 , 김연희 저 
- https://mangchhe.github.io/db/2022/01/22/PrimaryForeignKeyUpdate/
- https://vettabase.com/why-tables-need-a-primary-key-in-mariadb-and-mysql/
- https://stackoverflow.com/questions/7573590/can-a-foreign-key-be-null-and-or-duplicate