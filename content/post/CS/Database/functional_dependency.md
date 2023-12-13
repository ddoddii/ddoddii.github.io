+++
author = "Soeun"
title = "[데이터베이스] Functional Dependency"
date = "2023-11-01"
description = "함수 종속성의 정의"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
math = true
slug = "functional-dependency"
+++

## Functional Dependency
Functional Dependency : $x \rightarrow y$ 라고 쓰면, x 가 y 를 결정한다는 뜻이다. 즉, 아래 테이블에서 x 를 주면, y 를 알 수 있다는 뜻이다. 여기서 x 는 determinant 이고, y 는 dependent 이다. 

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
> $if \ t_{1}\cdot x = t_{2}\cdot x,\  then \ t_{1}\cdot y=t_{2}\cdot y$ 

$t_{1}$ , $t_{2}$ 는 각 row 를 나타낸다. 만약 x 값이 다르면, 더 이상 비교할 필요도 없다.
## Example 

이제 실제 테이블을 보자.

| Row_number | Name | Score | Dept | Course |
| ---------- | ---- | ----- | ---- | ------ |
| 1          | a    | 78    | CS   | C1     |
| 2          | b    | 60    | EE   | C1     |
| 3          | a    | 78    | CS   | C2     |
| 4          | b    | 60    | EE   | C3     |
| 5          | c    | 80    | IT   | C3     |
| 6          | d    | 80    | EC   | C2     | 

1.  R.No $\rightarrow$ Name (O). 왜냐면 Row_number 에 따라 Name 이 특정되기 때문이다. 
2. Name $\rightarrow$ R.No (X). 왜냐면 row1, row3 에서 Name 이 a 로 동일하지만, R.No 는 동일하지 않기 때문이다. 
3. R.No $\rightarrow$ Score (O)
4. Dept $\rightarrow$ Course (X). 왜냐면 row1, row3 에서 Dept 가 CS 로 같지만 Course 가 C1, C2 로 다르다.
5. Course $\rightarrow$ Dept (X)
6. R.No, Name $\rightarrow$ Score (O)
7. Name $\rightarrow$ Score (O). Name 이 같은 경우에도 Score 이 같다. 
8. (Name, Score) $\rightarrow$ Dept (O)
9. (Name, Score) $\rightarrow$ (Dept, Course) (X)


## Reference
- [Lec 4: Functional dependency in DBMS | What is functional dependency | Database Management System](https://www.youtube.com/watch?v=dR-jJimWWHA)

