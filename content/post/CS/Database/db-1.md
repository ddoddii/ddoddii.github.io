+++
author = "Soeun"
title = "[데이터베이스] Database and Database User"
date = "2023-09-11"
description = "Database 를 왜 쓰는가?"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = ""
+++

## Introduction
- (관계형) 데이터베이스 : 연관된 데이터의 모음 (collection of related data) ⭐️
- 데이터 : 의미를 갖는 사실 (known facts)
	- 전화번호, 주소, 주민등록번호
- 책의 한 페이지 : 관련된 데이터의 모임 -> 데이터베이스인가 ? 아니다 ! 
	- -> 내포된 성질 정의 필요 
- 관계형 데이터베이스의 성질 : Implicit properties
	- Universe of Discourse (UoD) : mini-world 
		- ex. 연세대학교 행정 데이터베이스 -> 연세대학교 내에서 mini-world 설정한다. 연세대, 서울대 데이터베이스는 mini-world 가 다르다. 
	- 고유의 의미를 갖고 논리적으로 결합된 데이터의 모임
	- 특정한 목적을 위해 설계, 구현됨 
- 데이터베이스 관리시스템(DBMS) : 데이터베이스를 정의, 생성, 유지하는 프로그램들의 집합
	- 데이터베이스 정의 : 데이터의 형, 구조, 제약조건 정의
	- 데이터베이스 구축 : 저장 매체에 데이터를 저장함 
	- 데이터베이스 조작 : 특정한 데이터의 질의 및 검색
- 데이터베이스 시스템 : <span style="background-color:#fff5b1"> DBMS + Application Program + Stored Database + Stored Database Definition </span>⭐️ 

	![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/975bd098-92ba-4420-8dc7-3b713448e3d7)
Base Table 마다 하나의 파일로 저장한다. 

## 데이터베이스의 정의
- 레코드 구조 정의 
	- 학생 파일 레코드 구조 - 이름, 학번, 학년, 전공
	- 각각 테이블의 레코드 구조를 정의해야 한다.
- 데이터 형 정의
	- 이름 : 문자열, 학번 : integer
	- 성적 : 문자 {A,B,C,D,F}
- 데이터베이스 구축 : 데이터의 저장
- 데이터베이스 조작 : 검색, 갱신, 첨가, 삭제 

## 데이터베이스 방법의 특징 
 <span style="font-size:120%"><span style="background-color: #EBFFDA">**정보를 관리하는 것은 파일로 처리할 수 있는데 굳이 데이터베이스를 왜 쓸까?**</span></span> 
- File processing <-> DBMS approach 
	- 만약 교무처에서 성적데이터 , 학생정보, 졸업학점을 교무처의 파일 시스템으로 관리하고, 재무처에서 등록금데이터, 학생정보를 재무처의 파일 시스템으로 관리한다고 하자. 어떤 학생이 학고 3번을 받아서 교무처에서 제적 처리 되었는데, 재무처에서는 이것을 모르고 등록금을 요구할 수 있다. 
	- 많은 동일한 데이터를 별도로 관리한다면, 저장 공간 및 관리 노력이 매우 비효율적이다. 
- DB System Approach
	- 성적데이터, 등록금데이터를 한 곳에 저장해놓고 교무처와 재무처에서 DBMS를 통해 성적 및 등록금을 입력/출력한다. 

<span style="font-size:120%"><span style="background-color: #EBFFDA">**DBMS의 특징**</span></span> 
- Self-describing nature 
	- DBMS 는 데이터를 설명하는 데이터(meta-data) / 실제 데이터  -> 2가지를 분리해서 저장
	- catalog : meta-data 를 저장하는 곳. 파일의 구조, 데이터의 형, 제약조건 등등을 작은 테이블 형태로 보관. 
	- DBMS와 사용자 참조
		- cf ) file processing : 데이터의 정의를 항상 응용프로그램에서 정의함 (application-dependent), 특정 응용프로그램에 종속적이고 프로그램간 공유 불가능 
- Insulation between Programs and Data 
	- program-data independence 
		- DBMS는 데이터 구조가 catalog 에 저장되어 독립적으로 관리
		- cf ) file processing : 데이터의 구조가 변경되면 프로그램도 변경됨
	- program-operation independence
		- DBMS system : object - oriented DBMS
		- file processing : 불가능
- Support of Multiple Views of Data ⭐️
	- **view** : 데이터베이스의 부분 또는 데이터베이스에서 유도된 가상 데이터
	- 실제로 저장된 base table 형태와 다르게, 각 사용자가 사용하는 데이터만을 가공해서 보여줌
- Sharing of Data and Multi-user Transaction Processing  ⭐️
	- 데이터는 공유 리소스이다. 여기에 동시에 접근하는 것을 제어. 가장 먼저 lock 을 가진(가장 먼저 접근한) 사람이 구매했다면 수량을 실시간으로 업데이트한다. 
	- Concurrency control (동시성 조절): control multiple simultaneous updates on the same data
	- ex. 비행기 예약, 콘서트 예약
## 데이터베이스 사용자군 
- DBA : DataBase Administrator
	- 데이터베이스 사용 허가 => 보안 유지(security)
	- 데이터베이스 사용 현황 조절 및 DB 성능 감시 => 성능 관리
- S/W & H/W 관리
- DB Designer
	- 데이터 정의 및 저장구조 설계 -> application domain expert
- End user : 데이터베이스를 사용하는 사람
	- casual end user :DB 사용 빈도수가 적지만 다양하고 복잡한 정보를 원하는 사용자 (중급/고급 관리자) (수강신청 관리자- 이 과목에 수강신청한 학생이 몇명인가?)
	- naive, parametric end user : 정형화된 질의/갱신 작업을 계속적으로 수행하는 사용자 (수강신청 하는 학생)
		- canned transaction : 정형화된 질의 갱신 작업
	- sophisticated end user : 데이터에 복잡한 작업을 수행하는 사용자 (엔지니어)
- 시스템 분석가 및 응용프로그램 개발자
	- 시스템 분석가 : 사용자의 요구사항을 분석 -> canned transaction 설계 (수강신청 절차 설계)
	- 응용프로그램 개발자 : canned transaction 구현, 디버깅, 시험

## DBMS 사용 목적
- Redundancy
	- file processing : 모든 파일을 독립적으로 보관 -> 동일한 데이터가 여러 데이터 파일에 저장됨
		- 저장공간 낭비
		- 동일한 데이터의 비일치 ❗️(두 파일에 저장된 데이터 중 한개만 갱신할 때 데이터 값이 일치하지 않음) 
	- DB system : 데이터 저장 공간 공유
		- 저장공간절약, 데이터 일치, 복사 및 관리노력의 감소
- Security
	- 데이터의 접근 허가 및 형태 관리 (read/write/privileged)
	- OS 에서는 파일 단위로 허가를 줌. DBMS 는 record 단위로 허가를 줌 ! 
- Multiple User Interface 
	- 다양한 사용자 유형 및 숙련도 지원
	- casual user : query language (SQL)
	- application programmer : programming language interface (Java)
	- naive user : natural language (미래에는 자연어 ..! )
- Represent complex relationship among data
- Enforcing integrity constraint (무결성 조건) ⭐️ <= semantics of DB  
	- ex. (제약조건) 학년 1-4, 과목번호 : unique, 성적 :{A,B,C,D,F}
	- 과목번호 CS3105 가 이미 있으면 동일한 과목번호는 더 이상 DBMS 에서 거부함. 성적이 E가 입력되면 DBMS 는 거부함. 
	- DBMS 에 의한 자동 점검 또는 데이터의 갱신 시 점검
- Backup & Recovery ⭐️
	- H/W 또는 S/W 고장 시 DBMS가 올바른 데이터로 유지
	- Recovery 기능을 DBMS가 모듈로 제공
	- Backup : 디스크를 복사해서 다른 디스크에 저장해놓는 것
		- 만약 backup 을 매월 말에 한다고 하자. 8월 31일에 디스크 backup 을 했는데, 9월 10일에 디스크가 날라갔다면 10일 치 데이터는 어떻게 되는걸까 ? DB에서는 모든 내역을 로그 파일에 저장해놓는다. 그래서 8월 31일 디스크와 로그파일을 가지고 10일치 내역을 다시 실행한다. 
	- Recovery  : 현재까지의 데이터를 복원하는 것
- 프로그램과 데이터 구조의 영구적 보관
	- cf ) file system : 프로그램이 파일과 메모리 사이의 데이터를 변환시킨다. 프로그램 실행 후에는 데이터는 파일에 보관되지만 변환이 필요하다.
	- DBMS : 데이터 구조는 meta-data 로 따로 저장하고, store-procedure 는 영구적으로 따로 저장할 수 있다. 

## DBMS 사용의 장점 
- 표준화
	- 명칭, 데이터 포맷, 출력 포맷 
	- cf ) file system : 각 파일의 총체적인 관리 불가능 
- 응용프로그램 개발 시간의 단축 
- 유동성 (유연성)
	- 새로운 변화에 대처 용이 
	- 기존의 데이터 / 프로그램에 영향 없음
- 최신 데이터 이용 가능 ⭐️
	- 다사용자 환경하에서의 concurrency control
- 경제성 : 중복 투자, 데이터 관리 인력 감소 

## DBMS 사용의 범위
- DBMS 사용 : overhead
	- 최초 투자 규모가 비싸다. 그렇지만 cloud 때문에 이 문제는 거의 사라졌다. 
	- overhead for security, concurrency control, recovery, integrity constraint checking
- When not to use DBMS
	- simple, well defined, not expected to change
	- real-time requirement : overhead 
	- no multiple user access


## Reference
- 2023-2 데이터베이스, 이원석 교수님