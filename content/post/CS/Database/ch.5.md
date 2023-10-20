+++
author = "Soeun"
title = "[데이터베이스] RDB and Relational Database contraints"
date = "2023-09-29"
description = "RDB 와 여러 제약조건들, key"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/35592c11-7971-472b-9668-3c4a2dbfa5e0"
math = true
+++

## 1. Relational Model Concepts

관계형 모델은 데이터베이스를 relation 의 모음으로 생각한다. 


### 1.1 Domains, Attributes, Tuples, and Relations

domain D 는 atomic value 들의 집합이다. atomic 하다는 것은 더 이상 나눌 수 없다는 의미이다. domain 을 정의하는 방법은 data type, name, format, additional info 를 구체적으로 정하는 것이다. 

예시) 이름 : 사람 이름의 집합 (유효한 사람이름을 나타내는 문자열의 집합), GPA : Floating point number 로, 0 <=GPA<= 4 의 범위 안에 있는 숫자. 

relational schema (R) 은 $R(A_1,A_2,...,A_n)$ 으로 표기된다. R 은 relational 이름 R 과 속성들 $A_1,A_2,...,A_n$ 의 리스트로 이루어져 있다. 각 **속성(attribute)** $A_i$ 는 도메인 D 에서 정한 구체적인 역할이다. D 는 **domain** of $A_i$ 라고 불리며, $dom(A_i)$ 로 표기한다. relational schema 는 relation 을 묘사하기 위해 사용된다. R 은 relation의 이름이다. relation 의 **degree** 는 n 으로 표시하는데, 이것은 속성의 수이다. 

예시) relation schema STUDENT -> STUDENT(Name, SSN, HomePhone, Address, Age, GPA)
- name of relation : STUDENT
- degree = 5
- dom(Name) = 학생 이름 집합
- dom(SSN) = 주민번호 집합 

domain 은 format, data type 만 정의하고, 실제 데이터를 포함하지는 않는다. 마치 변수를 정의하는 것과 같다. 

relation schema $R(A_1,A_2,...,A_n)$ 의 relation $r$ 은 $r(R)$ 로도 표기하고, 이것은 set of n-tuples 로, $r = \{t_1,t_2,...,t_m\}$ 이다. $t_i$ 는 n개 value 의 ordered list 로, $t_{i}=<v_1,v_2,...,v_n>$ 이다. 각 value $v_i$ 는 $dom(A_i)$ 의 요소이거나 NULL value 이다. $r(R)$ 은 relational schema $R$ 을 정의하는 모든 domain 의 cartesian product 의 부분집합이다. 

특정 시간에 relation state 를 current relation state 라고 한다. 즉, Mini world 의 특정 상태를 나타내는 유효한 tuple들의 모임이다. 

쉽게 말해, STUDENT relation 에 STUDENT<John, 998882772, 02-322-3049, nyc, 24, 3.8> 이 있을때, John 은 dom(Name) 의 요소에 해당하는 value 값이다. 이 행 자체가 하나의 튜플($t_i$)이 되는 것이다. 실제 데이터인 학생들의 데이터가 여러 행이 쌓여서 지금 시점의 current relational state 가 되는 것이다. 

### 1.2 Characteristics of Relations

#### Ordering of Tuples in a Relation 
relation 은 튜플들의 **집합(set)** 으로 정의한다. 집합은 순서가 없다. 따라서 relation 내에 있는 튜플들도 순서가 상관이 없다. 반면에 file 은 물리적으로 디스크에 저장된다. 따라서 file 은 수서가 존재한다. 

#### Ordering of Values within a Tuple and an Alternative Definition of a Relation 
n-tuple 은 n개 value 의 *ordered list* 라고 정의했다. 하지만 사실 속성의 순서는 중요하지 않다. 따라서, 튜플의 순서를 무시할 수 있는 대안할만한 정의를 제시한다. 여기서 relational schema $R=\{A_1,A_2,...,A_n\}$ 는 속성들의 **집합**이다. 그리고, relational state $r=\{t_1,t_2,...,t_m\}$ 으로, $t_i$ 는 R -> D 로 mapping 이다. D 는 attribute domain 의 union 이다. $D=dom(A_{1}) \cup dom(A_{2}) \cup...\cup dom(A_n)$ 이다. 이 정의에선, $t[A_i]$ 는 $dom(A_i)$ 내에 존재해야 한다. 

이러한 정의 아래에서는, tuple 은 ([attribute],[value]) 의 집합으로 생각될 수 있다. 여기서는, 튜플의 순서가 중요하지 않게 된다. 

#### Values and NULLs in the Tuples
튜플 내에 있는 value 는 atomic 하다. 따라서, composite 하거나 multi-valued 속성은 없다. 

또 다른 중요한 컨셉은 NULL value 이다. NULLvalue 는 value unknown, exists but is not available, attribute does not apply to this tuple(value undefined) 할 때 사용된다. NULL 이라고 존재하지 않는 값은 아니란 말이다 ! 

하지만 최대한 NULL 값이 존재하지 않도록 데이터베이스를 설계하는 것이 좋다. 

#### Interpretation of a Relation 
relational schema 는 declaration 또는 assertion 으로 해석될 수 있다. 즉, relationship 과 fact 의 uniform representation 이다. 엔티티에 대한 fact 이고, 엔티티간의 관계 (relationship) 을 정의한다. 


## 2. Relational Model Constraints and Relational Database Schemas

### 2.1 Domain Constraints
Domain constraints 는 각 튜플 내에서, attribute A 의 value 는 atomic value 여야 한다는 것이다. 각 domain 마다 data type 을 명시한다.

### 2.2 Key Constraints and Constraints on NULL Values
relation 은 set of tuples 라고 했다. 집합에서는 모든 원소가 다 달라야 한다. 따라서, relation 내에 있는 tuple 들도 다 달라야 한다. 여기서 몇개의 속성을 뽑아서 그 속성들만 가지고 모든 튜플을 비교해봐도 겹치는 것이 없는 속성들이 있다. 

 $t_{1}[SK]\neq t_2[SK] \ where \ t_{1} , t_{2} \notin r(R)$ 을 만족하는 속성 SK를 relational schema R 의 **superkey** 라고 한다. 모든 relational schema 는 적어도 한 개 이상의 superkey 를 가지고 있다. superkey 중 minimal 한 조건을 만족하는 것을 **key** 라고 한다. 즉 superkey 이면서, 어느 속성을 제거했을 때 더 이상 superkey 가 안되는 superkey 가 key 이다.  key 는 아래 2개의 조건을 만족해야 한다. 
 
 1. uniqueness
 2. minimal superkey 

relational schema 는 한 개 이상의 Key 를 가질 수 있다. 이때, 각 key 들을 **candidate key** 라고 한다. 데이터베이스 설계자는 이 candidate key 중에서 **primary key** 를 고른다. 이때 단순속성이거나, 최대한 적은 수의 속성을 갖는 복합속성을 선택한다. 

더 자세한 것을 [RDB 의 key](https://ddoddii.github.io/post/cs/database/key/#super-key) 에서 따로 다루었다. 

### 2.3 Relational Databases and Relational Database Schemas
relational database 는 여러 개의 relation 으로 구성된다. relational database schema(S)는 relational schema(R) 의 집합으로,  $S=\{R_{1},R_{2},...,R_m\}$  와 integrity constraints(IC) 로 이루어진다. relational database instance DB 는 integrity constraint 를 만족하는 relation instance 의 집합이다. 

relational database = relational database schema(데이터베이스 설계도) + relational database instance(실제 정보)


### 2.4 Entity Integrity, Referential Integrity, and Foreign Keys 

#### Entity Integrity
Primary Key 의 value 는 NULL 이 되면 안된다는 제약조건이다. PK 는 relation 의 튜플을 구분하는데 사용된다. 따라서 NULL 값이 오면 더 이상 PK를 가지고 튜플들을 구분할 수 없을 것이다. 

#### Referential Integrity
2개 Relation 사이에 동일성을 보장하기 위해 사용되는 제약조건이다. 하나의 Relation 튜플에서, 다른 relation 을 참조할 때, 그 relation 에 존재하는 튜플을 참조해야 한다는 조건이다. 이것을 이해하려면 Foreign key 에 대한 더 깊은 이해가 필요하다. 

#### Foreign Key
relation schema $R_1$ , $R_2$ 가 있다고 하자. $R_1$ 의 FK 가 $R_2$ 를 참조할 때, 아래 2개의 규칙을 만족해야 한다.
1. FK 가 가지고 있는 속성들은 $R_2$ 의 PK 의 속성의 domain 과 같은 domain 이어야 한다. 
2. 현재 상태 $r_1(R_1)$  tuple $t_1$ 의 FK 의 value 는 $r_2(R_2)$ 상태의 튜플 $t_2$ 의 PK 값이거나, NULL 값이어야 한다. ,  $t_1[FK]=t_2[PK]$ or NULL 이어야 한다. 


## 3. Update Operations, Transactions, and Dealing with Constraint Violations

### 3.1 Insert Operation 
Insert 는 새로운 tuple 을 relation 에 첨가하는 것이다. 이때 체크해야할 제약조건은 4가지이다. 
- domain constraint

  첨가되는 tuple 에 있는 속성의 값이 속성의 domain 에 없을 때
- key constraint

  첨가되는 tuple 의 key 값이 relation 의 다른 tuple 의 key 값으로 존재할 때
- entity constraint

  첨가되는 tuple 의 PK 값이 NULL 일 때
- referential constraint

  참조되는 relation 에 첨가되는 tuple 의 foreign key 값을 갖는 tuple 이 존재하지 않을 때

constraint 를 어기면, insertion 을 거부하거나, 유저와 함께 insertion 을 고치는 작업을 한다. 
### 3.2 Delete Operation 
Delete 는 referential integrity constraint 만 체크하면 된다. 삭제될 tuple 이 다른 relation 에 의해 참조되는 경우 조심해야 한다. 

대처방안으로는, delete 자체를 거부하거나, 삭제된 tuple 을 참조하는 모든 tuple 을 삭제할 수 있다. 혹은 이 tuple 을 참조하는 모든 값을 NULL 또는 다른 값으로 대체할 수 있다. 하지만 이 참조하는 값이 PK 면 NULL 로 대체하면 안된다. 보통 3가지 방안을 혼합해서 사용한다. 

### 3.3 Update Operation 
Update 는 delete + insertion 연산이다. 따라서 앞서 봤던 delete, insertion 의 제약조건들을 모두 확인해야 한다. 

## Reference
- Fundamentals of database systems, 7th edition, ch.5
- 2023-2 데이터베이스, 이원석 교수님