+++
author = "Soeun"
title = "[데이터베이스] Enhanced Entity-Relationship(EER)"
date = "2023-09-23"
summary = "superclass/subclass, specialization, union type"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
math = true
series = ["Database"]
series_order = 4
+++

## 1. Subclasses, Superclasses, and Inheritance
EER 은 ER 모델과 더불어서, subclass / superclass 개념도 다룬다. 이것은 모델링을 할 때 항상 나오는 specialization / generalization 에 관련한 개념이다. 또 다른 컨셉은 category / union type 이다. 그리고, attribute 과 relation 에 대한 상속도 다룰 것이다. 

### subtype / subclass
예를 들어, Employee 라는 엔티티가 있다고 하자. Employee 에도 secretary, engineer, manager 등 여러 종류가 있다. 이 세분화된 엔티티를 employee 의 subclass 라고 한다. 그리고 상위 클래스인 employee 를 superclass 라고 한다. 

If class $C_1$ is a super class of class $C_2$ ,
let $e_{i} \in C_{1}$ and $e_{j} \in C_{2}$ $\longrightarrow$ $e_{j} \in C_{1}$ but $e_{i} \notin C_{2}$ 

이 superclass 와 subclass 간의 관계를 class/subclass relationship 이라고 한다. 이것은 IS-A 관계이다. 즉, subclass 에 속한 엔티니는 superclass 의 멤버이기도 한다. 또, 중요한 개념은 type inheritance 이다. superclass 에 속한 속성들을 subclass 는 상속받는다. 

## 2. Specialization and Generalization


**Specialization** 은 엔티티 타입의 set of subclass 를 정의하는 과정이다. 즉, 상위 개념을 하위 세부적인 개념으로 나누는 과정이다. subclass 에만 속하는 속성을 specific attribute 라고 한다. 예를 들어, Employee 에게는 없고 engineer 에게만 있는 'skill' 속성을 specific attribute 이라고 한다. subclass 는 또 specific relation type 에 참여할 수 있다. engineer 만 'TECH_UNION' 에만 'BELONGS_TO' 관계를 가지는것이 이것에 해당한다. 

specialization 을 하는것에는 2가지 이유가 있다. 
1. 특정 속성이 몇몇의 엔티티에만 적용되고, 모두에게 적용되지 않을 때가 있다. secretary, engineer, manager 모두 employee 의 공통 속성인 salary 를 공유하지만, 각각의 직군 마다 가지는 속성이 다르다. 
2. 관계가 특정 엔티티에만 적용될 수 있다. 

Generalization 은 specialization 과 반대의 개념이다. 

## 3. Constraints and Characteristics of Specialization and Generalization Hierarchies

### 3.1 Constraints on Specialization and Generalization 

superclass 에 제약조건을 주어서, subclass 에 속한 엔티티들을 정확하게 정의할 수 있다. 이러한 subclass 들을 **predicate-defined** 또는 condition-defined subclass 라고 한다. 위의 예시에서는 , Employee 가 Job_type 에 따라 분류가 되었다. 이때 job_type 가 constraint 가 되는 것이다. 

수학적으로는 , S (Superclass) , C(Subclass) 라 하면, $S=C[P]$ 라 나타낼 수 있다. P 가 defining predicate 가 되는 것이다. 

<img width="579" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/31b523e1-4ad2-4dcd-b8dc-6e9a734364f6">

다른 제약조건들로는 **disjoint constraint** 가 있다. disjoint 제약조건이 없는 경우를 **overlapping** 이라고 한다. 
- disjoint subclass

  하위 subclass 들은 모두 disjoint set 이어야 한다. 즉, 엔티티는 오직 1개의 subclass 의 멤버만 될 수 있다. 원 안의 d 가 disjoint 의 약자이다. 
  $S_{i}\cap S_{j}= \phi$  for $i\neq j$ 
- overlapping subclass

  엔티티는 1개 이상의 subclass 의 멤버가 될 수 있다. 이것은 원 안에 o 를 쓰는 것으로 나타낸다. 

또 다른 제약조건에는 completeness constraint가 있다. 
- total specialization 

  superclass 내에 있는 모든 엔티티는 꼭 subclass 중 하나의 멤버여야 한다. 다이어그램에서는 더블라인으로 나타낸다. 
- partial specialization

  엔티티가 어느 subclass 에 속하지 않을 수 있다. 

위 2가지 제약조건은 독립적이다. 따라서 4가지 종류가 있을 수 있다.
- disjoint, total
- disjoint, partial
- overlapping, total
- overlapping, partial

### 3.2 Specialization and Generalization Hierarchies and Lattices

**specialization hierarchy** 는 subclass가 오직 1개만의 class/subclass 관계에만 참여할 수 있다는 제약조건이 있다. 즉, single inheritance 이다.  이것으로는 tree 구조가 나온다. 

반면에 , **specialization lattice** 는 2개 이상의 class/subclass 관계에 참여할 수 있다. 즉, multiple inheritance 이다. 하나 이상의 superclass 를 가진 subclass 를 shared subclass 라고 한다. 아래의 사진에서 Engineer 는 Employee , Engineering_manager 2가지 클래스를 동시에 상속받는다. 이것은 engineer 는 상위 두 클래스에 모두 IS-A 관계가 성립하는 것이다. 

<img width="550" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7dfc5808-3c26-4f69-9e97-85e15b13485b">


## 4. Modeling UNION types using Categories

다른 엔티티를 대표하는 엔티티를 만들 필요성이 생기기도 한다. subclass 가 서로 다른 엔티티 타입의 집합(union) 를 대표할 때, 이 subclass 를 **union type** 혹은**category** 라고 부른다.  

예를 들어, 3가지 엔티티 타입(person, bank, company) 가 있다고 하자. 자동차 등록 데이터베이스를 만들 때, 자동차 주인(owner)의 역할을 할 클래스를 만들어야 한다. category OWNER 는 3개 엔티티 타입(person, bank, company) 의 UNION 의 subclass 이다. 

union type 과 아까 다룬 shared subclass 가 헷갈릴 수 있는데, 비교해보자. 위에 예시에서 ENGINEERING_MANAGER 는 shared subclass 이고, 아래 사진에서 OWNER 는 union type 이다. 

<img width="340" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/2ddee2b6-4cba-4fcc-9e1b-783491974c1f">

|  **union type**   | **shared subclass**    |
| --- | --- |
| "OR" 관계이다. 서로 다른 엔티티 타입을 '대표' 하는 것이 카테고리이다.<br> 위에서, OWNER 는 하나의 슈퍼클래스로만 존재해야 한다. 즉, OWNER 는 PERSON 또는 BANK 또는 COMPANY 중 1개여야 한다.     | subclass 는 모든 superclass 에 존재해야 한다. 즉 2개 이상의 슈퍼클래스가 있으면, 상위의 슈퍼클래스에 모두 IS-A 관계여야 한다. <br> 위에서 ENGINEERING_MANAGER 는 ENGINEER, MANAGER, SALARIED_EMPLOYEE 3개 모두여야 한다.    |

category 도 total 또는 partial 일 수 있다. total category 는 모든 union 을 나타내고, partial 는 subset of union 을 나타낸다. 즉, total 은 owner 가 속할 수 있는 모든 엔티티를 다 펼쳐놓는 것이고, partial 은 그 중 부분만 제시하는 것이다. 

## Reference
- Fundamentals of database systems, 7th edition, ch.4
- 2023-2 데이터베이스, 이원석 교수님