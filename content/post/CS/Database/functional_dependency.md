+++
author = "Soeun"
title = "[데이터베이스] Functional Dependency and Closure Set"
date = "2023-10-27"
summary = "함수 종속성의 정의와 Closure"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
math = true
slug = "functional-dependency"
series = ["Database"]
series_order = 7
draft = true
+++
{{< katex >}}

## Functional Dependency
Functional Dependency : \\( x \rightarrow y \\) 라고 쓰면, x 가 y 를 결정한다는 뜻이다. 즉, 아래 테이블에서 x 를 주면, y 를 알 수 있다는 뜻이다. 여기서 x 는 **determinant** 이고, y 는 **dependent** 이다. 

| x   | y   |
| --- | --- |
| x1  | y1  |
| x2  | y2    |

구체적인 예시를 보자. 

| x   | y   |
| --- | --- |
| 1   | 1   |
| 2   | 1   |
| 3   | 2   |
| 4   | 3   |
| 2   | 5   |

x 가 3 일 때, y는 2이다. 하지만 x 가 2일때, y 는 1 일수도 있고 5일 수 도 있다. 따라서 x, y 는 Funciontal dependent 관계가 아니다. x 를 알아도 y 를 특정할 수 없다. Funciontal dependent 관계를 만족하려면, x 값이 동일하면 y 값도 동일해야 한다 .

Functional dependency 에 대한 정의를 보자 .
> \\(if \ t_{1}\cdot x = t_{2}\cdot x,\  then \ t_{1}\cdot y=t_{2}\cdot y\\)

\\(t_{1} , t_{2}\\) 는 각 row 를 나타낸다. 만약 x 값이 다르면, 더 이상 비교할 필요도 없다.

이제 실제 테이블을 보자.

| Row_number | Name | Score | Dept | Course |
| ---------- | ---- | ----- | ---- | ------ |
| 1          | a    | 78    | CS   | C1     |
| 2          | b    | 60    | EE   | C1     |
| 3          | a    | 78    | CS   | C2     |
| 4          | b    | 60    | EE   | C3     |
| 5          | c    | 80    | IT   | C3     |
| 6          | d    | 80    | EC   | C2     | 
1.  R.No \\(\rightarrow\\) Name (O). 왜냐면 Row_number 에 따라 Name 이 특정되기 때문이다. 
2. Name \\(\rightarrow\\) R.No (X). 왜냐면 row1, row3 에서 Name 이 a 로 동일하지만, R.No 는 동일하지 않기 때문이다. 
3. R.No \\(\rightarrow\\) Score (O)
4. Dept \\(\rightarrow\\) Course (X). 왜냐면 row1, row3 에서 Dept 가 CS 로 같지만 Course 가 C1, C2 로 다르다.
5. Course \\(\rightarrow\\) Dept (X)
6. R.No, Name \\(\rightarrow\\) Score (O)
7. Name \\(\rightarrow\\) Score (O). Name 이 같은 경우에도 Score 이 같다. 
8. (Name, Score) \\(\rightarrow\\) Dept (O)
9. (Name, Score) \\(\rightarrow\\) (Dept, Course) (X)

## Types of Functional Dependency

### Trivial Functional Dependency

> x \\(\rightarrow\\) y if y \\(\subseteq\\) x

위의 예시에서, (R.No, Name)  \\(\rightarrow\\) Name 은 **Trivial functional dependency** 이다. Name 이 (R.No, Name) 의 집합에 속하는 원소이기 때문이다. Name  \\(\rightarrow\\) Name 도 trivial fd 이다. Trivial FD 는 항상 성립한다. 

### Non-Trivial Functional Dependency

> x  \\(\rightarrow\\) y , x \\(\cap\\) y = \\(\emptyset\\) 

위의 예시에서, R.No  \\(\rightarrow\\) Name 은 Non-Trivial dependency 이다. 이 경우에는 실제로 성립하는지 테이블에서 확인해야 한다. 

### Semi-Trivial Functional Dependency

> x \\(\rightarrow\\) y , y \\(\notin\\) x

위의 예시에서, (R.No, Name) \\(\rightarrow\\) (Name, Score) 는 Semi-Trivial Functional Dependency 이다. Name 은 공통이지만, y 가 x 에 속하지는 않기 때문이다.

## Armstrong's Axioms / Inference Rules

Armstrong's Axioms 를 사용하여 테이블에 존재하는 FD 를 모두 알 수 있다. 

1. **Replexivity** : x\\(\rightarrow\\) x , x \\(\rightarrow\\) y if y \\(\subseteq\\) x . 
2. **Transitivity** : if (x \\(\rightarrow\\) y & y \\(\rightarrow\\) z) then x \\(\rightarrow\\) z. 
3. **Augmentation** : if x \\(\rightarrow\\) y then xA \\(\rightarrow\\) yA 
4. **Union** : if (x \\(\rightarrow\\) y & x \\(\rightarrow\\) z) then x \\(\rightarrow\\) yz
5. **Decomposition** : if x \\(\rightarrow\\) yz then x \\(\rightarrow\\) y & x \\(\rightarrow\\) z (오른쪽(dependent)만 split 할 수 있다 !!)
6. **Pseudotransitive** : if (x \\(\rightarrow\\) y & yz \\(\rightarrow\\) A) then xz \\(\rightarrow\\) A  
7. **Composition** : if (x \\(\rightarrow\\) y & A \\(\rightarrow\\) B) then xA \\(\rightarrow\\) yB


## Attribute Closure

R(A,B,C,D,E)

Functinal Dependency : {A\\(\rightarrow\\) B, B\\(\rightarrow\\)C,C\\(\rightarrow\\)D,D\\(\rightarrow\\) E}

- A \\(\rightarrow\\) C (O) : Transitivity property 때문에 맞다.
- A \\(\rightarrow\\) A (O) : Replexivity property
- A\\(\rightarrow\\) D (O)
- A\\(\rightarrow\\) E (O)
- A \\(\rightarrow\\) {A,B,C,D,E} (O): Union property
- AD \\(\rightarrow\\) BD (O) : Argumentation property
	- AD \\(\rightarrow\\) B , AD \\(\rightarrow\\) D : Decomposition 

\\(X\\) : Set of arrtibutes 

\\(X^{+}\\) : contains set of attributes determined by \\(X\\) 

위의 예시를 다시 보자,
- \\(A^+\\) = {A,B,C,D,E} 
- \\(AD^+\\) = {A,D,B,C,E} 
- \\(B^+\\) = {B,C,D,E}
- \\(CD^+\\) = {C,D,E}

여기서 슈퍼키를 찾을 수 있다. 슈퍼키의 closure는 R 의 모든 속성을 포함한다. 따라서 위의 예시에서는 A, AD, AB , ... 가 슈퍼키이다. 슈퍼키는 몇개일까? A 를 포함하는 모든 집합이 될 것이다. A 를 제외한 combination 의 수는 \\(2^4=16\\) 이다. 따라서 슈퍼키는 16개이다. 

후보키는 슈퍼키 중에서 최소성을 만족하는 키이다. 따라서 A 만 후보키가 된다. 

## Minimal Sets

A FD set F is mininal if, 
1. for \\(\forall\\) FD's X \\(\rightarrow\\) Y in F, Y is a single attribute
 2. Let E = (F - one of FD's in F) \\(\rightarrow\\) \\(E^{+}\neq F^+\\) 
3.  Let E = (F - {X \\(\rightarrow\\) A}) \\( \cup \\) Y \\(\rightarrow A\\) where Y \\(\subseteq\\) X \\(\rightarrow\\) \\(E^{+}\neq F^+\\) 

\\(\longrightarrow\\) \\(F_{min}=F\\) 



## Reference
- [Lec 4: Functional dependency in DBMS | What is functional dependency | Database Management System](https://www.youtube.com/watch?v=dR-jJimWWHA)
- [Lec 5 : Armstrong's axioms in DBMS | Inference rules of Functional Dependency](https://www.youtube.com/watch?v=eIH7zRVelnw)
- [Lec 6 : What is Attribute Closure |Closure set of Attribute in DBMS | How to find Closure of Attributes](https://www.youtube.com/watch?v=AGFUfLPFJ7w&list=PLdo5W4Nhv31b33kF46f9aFjoJPOkdlsRc&index=6)

