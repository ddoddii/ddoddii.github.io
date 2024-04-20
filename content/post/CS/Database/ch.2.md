+++
author = "Soeun"
title = "[데이터베이스] Database System Concepts and Architecture"
date = "2023-09-17"
summary = "Database 설계에 대한 역사"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
series = ["Database"]
series_order = 2
draft = true
+++

## Data Models, Schemas, and Instances

### Categories of data models

데이터 모델의 종류에는 3가지가 있다. 

**개념적 데이터 모델(High-level / Conceptual data model)**
- DB 사용자가 이해하는 데이터베이스의 구조이다. 
- 개체(entity) : 실세계의 물건, 개념 
- 속성(attribute) : 개체의 성질, 속성
- 관계(relationship) : 둘 이상의 개체간의 관계 

**물리적 데이터 모델(low-level / Physical data model)** 
- 데이터의 저장 방법을 기술한다. 
- 레코드의 포맷/ 순서를 결정한다. 
- 데이터 접근 경로(각 레코드를 찾아가는 포인터 구조) 를 설정한다.

**구현 데이터 모델(representational / implementation data model)**
- 데이터베이스 관리 시스템(DBMS) 에서 사용하는 모델로, 개념적 데이터 모델과 물리적 데이터 모델의 중간 모델이다. 
- 물리적인 상세한 정보를 추상화 하면서 데이터의 구조를 표현한다. 

### Schemas, Instances, Database State

A **data model** is a collection of concepts for describing data. A **schema** is a description of a particular collection of data, using a given data model. The **relation model of data** is the most widely used model today. 

data model 에서는 데이터베이스를 설명하는 하는 데이터(schema)와, 데이터베이스 자체를 구분해야 한다 !! 

Database schema (meta-data)
- 데이터베이스에 대한 설명 : 실제 저장 데이터 제외
- DBMS 의 catalog 에 저장됨
- 데이터베이스 설계 시 정의되며 자주 변경되지 않음
- 스키마 도표 : 스키마를 표현하는 그림
- 제한된 정보 표현 

Instance - 데이터베이스에 실제로 저장된 데이터

database states
- a set of instances at a particular moment
- 특정 시점에서 데이터베이스에 저장된 데이터들의 집합
- 레코드 첨가 , 삭제 등 -> 데이터베이스의 상태를 변화시킨다. 

새로운 database 를 정의할 때, 우선 database 의 schema 를 설계한다. 이 시점에서 database state 는 empty state 이다. 아직 아무런 실제 instance 가 저장되지 않았기 때문이다. DBMS는 데이터베이스에 변화가 생길 때마다, 스키마에 정의된 제약조건을 가지고 데이터베이스가 항상 valid state 인지 검사한다. DBMS 는 스키마에 대한 정의와 제약조건 (meta-data) 을 catalog 에 저장하고, state 를 검사할 때마다 여기서 조건들을 확인한다. 

## Three-schema Architecture and Data Independence

앞서서 데이터베이스를 사용하는 이점들에는 (1) catalog 를 사용해서 데이터베이스 설명(schema) 를 저장해서 self-describing 하게 하는 것 , (2) insulation of program and data, (3) support of multiple views 가 있다고 했다. 여기서는 이것들을 가능하게 하는 DBMS 구조 , 즉 three-schema architecture 에 대해 보고자 한다. 

### Three-Schema Architecture


<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fbb0133d-7b4e-46d4-835c-771b1a4969a3">

**Internal schema**
- 물리적 데이터 모델 사용해서 데이터베이스의 물리적 저장 구조를 기술함

**Conceptual schema**
- 유저들을 위해 데이터베이스의 전체 구조 기술
- 물리적인 저장 구조 방식은 숨기면서, entities, data types, relationships, user operations, constraints 를 기술하는데 집중함 
- 주로 구현 데이터 모델을 사용하는데, 이것은 개념적 데이터 모델을 바탕으로 하고 있다. 

**External schema**
- 사용자의 필요에 따라 데이터베이스의 일부분을 기술한다. 
- 각 external schema 는 특정 유저 그룹이 관심 있는 데이터베이스의 특정 부분을 나타낸다. 따라서 유저의 특성에 맞게 여러 개의 view 가 존재할 수 있다.
- 주로 구현 데이터 모델을 사용하는데, 이것은 개념적 데이터 모델을 바탕으로 한다. 

3개의 스키마가 있지만, 이것은 데이터에 대한 설명일 뿐이다. 실제 데이터는 오직 물리적 레벨에만 저장된다. 특정 유저가 external schema 에 요청을 날리면, DBMS는 이 요청을 conceptual schema 에 맞게 변환한다. 그 다음 이 요청은 internal schema 에 맞게 변환해서 실제 데이터를 다룬다. 각 레벨 간에 요청을 변환하는 것을 mapping 이라고 한다. 

### Data Independence

데이터 독립성은 하나의 레벨에 있는 스키마를 변경하면서, 그 상위 레벨에 있는 스키마를 변경하지 않아도 되는 것이다. 

**Logical Data Independence**
- conceptual schema 를 변경하면서, external schema 또는 application program 을 변경하지 않아도 되는 것이다. 
- 우리는 레코드 타입을 추가하거나, 데이터 아이템을 추가함으로써 conceptual schema을 변경할 수 도 있다. 이때, view definition 과 mapping 만 변경하면 된다. 

**Physical Data Independence**
-  conceptual schema 와 external schema 에 영향을 주지 않으면서, internal schema 를 변경할 수 있는 것이다. 
- 즉 물리적인 파일을 수정해도, 외부스키마를 사용하는 응용프로그램에는 영향이 없는 것이다. 

##  Database Languages and Interfaces

### DBMS Languages

스키마 정의 언어에는 3가지가 있다. 

**Data Definition Language(DDL)**
- 개념적 스키마 및 매핑을 정의한다. 
- 설계된 DB 구조를 DBMS 의 catalog 에 입력한다. 
- 각 레벨간의 명확한 경계가 없다면 DDL 만 사용해서 모든 작업을 할 수 있다. 즉, DDL 이 SDL 와 VDL 다 포괄한다. 

**Storage Definition Language(SDL)**
- 내부와 개념 스키마의 정확한 구분이 있는 경우, 내부 스키마 및 매핑을 정의한다. 

**View Definition Language(VDL)**
- 외부 스키마(view) 및 매핑을 정의한다. 

데이베이스 스키마가 정의되고, 실제 인스턴스들이 데이터베이스에 저장되면, 데이터베이스를 조작(검색, 삽입, 삭제, 갱신) 하기 위한 언어가 필요하다. 이때 사용하는 것이 DML 이다. 

**Data Manipulation Language(DML)**
- 대부분 통합된 형태의 언어를 사용한다. (SQL = DDL + DML)
- 2가지 Type 의 DML 이 있다. 
	- High-level DML : 단독 사용해서 복잡한 데이터베이스 연산을 할 수 있다. 터미널을 통해 직접 입력할 수 있거나, 프로그래밍 언어에 삽입되어 있다. SQL 이 여기에 해당한다. SQL은 여러 개의 레코드를 한번에 가져올 수 있으므로 set-at-a-time DML 이라고 불린다. 어떤 데이터를 가져올 지 명시하므로, declarative 언어로 불린다. 
	- Low-level DML : 프로그래밍 언어에 삽입되어야 한다. 각 레코드를 가져올 때 연산을 각각 해야 한다. 그래서 recode-at-a-time DML 로 불린다. 레코드를 어떻게 가져올 지 명시해줘야 한다. 
- DML 이 상위 프로그래밍 언어에 삽입되어 있는 형태이면, 이 언어는 host language 이고, DML 은 data sublanguage 이다. 반면에, high-level DML 이 혼자서 명령을 내리면 query language 라고 부른다. 
- casual end user 는 high-level query language 를 사용하고, programmer 는 DML 을 embedded form 형태로 사용한다. 

## Database System Environment

### DBMS Component Modules

데이터베이스와 DBMS catalog 는 디스크에 저장된다. 디스크에 접근하는 것은 디스크의 read/write 를 스케쥴링하는 os 에 의해 제어된다. 많은 DBMS 는 디스크 read/write 를 스케쥴링하는 자기만의 buffer management module 을 가지고 있다. 디스크에 읽기 , 쓰기를 줄이는 것은 퍼포먼스를 굉장히 향상시킨다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/46a8afc6-3233-4cc2-b887-ddf55f1d8b45">

## Centralized and Client/Server Architectures for DBMS
### Centralized DBMS Architecture

DBMS 의 설계는, 컴퓨터 시스템 설계와 비슷한 트렌드를 따라갔다. 옛날의 설계는 mainframe 컴퓨터를 사용해서 모든 시스템 기능들(DBMS 기능, 유저 어플리케이션 프로그램, 유저 인터페이스 등등..)을 제공했다. 하지만 하드웨어의 가격이 떨어지자, 유저는 터미널들을 PC 와 workstation 들로 교체했다. 처음에는, DBMS 도 centralized 된 구조였다. 하지만 시간이 지남에 따라 DBMS도 client/server DBMS 설계를 도입하기 시작했다. 

### Basic Client/Server Architectures

클라이언트-서버 구조는 네트워크를 통해 여러 명의 클라이언트와 서버들이 연결되어 있는 환경을 위해 나타냈다. 아이디어는 특정한 기능을 가진 서버를 정의하는 것이었다. 예를 들어 file server / printer server / Web server 이렇게 기능을 나누는 것이다. 그러면 클라이언트 머신은 적절한 인터페이스를 유저에게 제공한다. 클라이언트는 유저 인터페이스와 로컬 프로세싱 능력을 가진 유저의 머신이다. 클라이언트는 자신에게 없는 추가적인 기능(DBMS 접속 등) 을 위해 이 기능을 제공하는 서버와 연결한다. 서버는 클라이언트에게 서비스를 제공해주는, 하드웨어와 소프트웨어가 있는 시스템이다. 

여기서 2가지 타입의 DBMS 설계가 만들어졌다. : two-tier & three-tier

### Two-Tier Client/Server Architectures for DBMS

RDBMS 에서, 클라이언트 쪽으로 가장 먼저 옮겨진 시스템 구성요소는 유저 인터페이스와 어플리케이션 프로그램이었다. SQL 이 RDBMS 를 위해 표준 언어를 제공하자, 이것은 클라언트와 서버를 나누는 논리적인 구분점이 되었다. 쿼리와 SQL 처리는 서버 쪽에 남았다. 그래서 서버는 query server 또는 transaction server , SQL server 로도 불린다. 

유저가 DBMS 접속이 필요하면, 클라이언트 머신에 있는 어플리케이션 프로그램은 DBMS 에 연결을 요청한다. 연결이 수립되면, 프로그램은 DBMS 와 소통할 수 있다. Open Database Connectivity (ODBC) 는 클라이언트 쪽이 DBMS 를 콜 할 수 있는 API 를 제공한다. Java 에서는 JDBC 가 이 기능을 제공한다. 

여기 있는 구조는 two-tier architecture 를 설명한다. 왜냐하면 소프트웨어 구성요소가 client / server 2개에만 있기 때문이다. 

### Three-Tier and n-Tier Architectures for Web Applications

많은 웹 어플리케이션은 three-tier architecture 를 사용한다. 

<img width="438" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/49e9cef8-8378-466d-830c-e16530177c4f">

middle tier 에 있는 레이어는 application server / web server 로 불린다. 여기서는 클라이언트에게서 요청을 받아서, 요청을 처리하고 , 적절한 데이터를 받아오기 위해 데이터베이스에 쿼리를 날린다. 따라서 _user interface, application rules, data access_ 는 three-tier 로 작동한다. 

n-tier architecture 도 가능한데, 보통은 비즈니스 로직 레이어를 또 여러 개의 레이어로 나눈다. 

## Classification of Database Management Systems

데이터 모델 분류
- relational, network, hierarchical, object-oriented .. 

사용자 수에 따른 분류 
- single-user : 하나의 유저만 접속한다. 
- multiuser : 동시에 여러 유저가 접속한다. 

데이터베이스가 분산된 사이트 수 
- centralized : 한 컴퓨터에 DBMS 와 모든 database 가 있다. 
- distributed : 네트워크로 연결된 여러 개의 사이트에 DBMS 와 데이터베이스가 분산된다. 
	- homogeneous DBMS : 모든 사이트가 동일한 DBMS 사용
	- heterogeneous DBMS : 사이트 마다 다른 DBMS 사용

데이터베이스 사용 목적 
- general - purpose DBMS
- special - purpose DBMS

## Reference
- Fundamentals of database systems, 7th edition
- 2023-2 데이터베이스, 이원석 교수님