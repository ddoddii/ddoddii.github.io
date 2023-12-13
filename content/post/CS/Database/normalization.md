+++
author = "Soeun"
title = "[데이터베이스] 정규화의 의미 - 1NF, 2NF, 3NF, 4NF, 5NF"
date = "2023-10-27"
description = "정규화는 무엇이고, 왜 해야 할까?"
categories = [
    "CS"
]
tags = [
    "데이터베이스"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/9246b7a7-030a-4057-96fd-28b1ce6b2c13"
math = true
slug = "normalization"
+++

Normalization(정규화) 는 무엇이고, 왜 해야할까?


훌륭한 데이터베이스는, 논리적으로 말이 안되는 데이터들이 저장되는 것을 방지해야 한다. 예를 들어, 하나의 손님이 2개의 서로 다른 생일 정보를 가질 수 없다. 이렇게 데이터 끼리 동의하지 않는 것을 'failure of data integrity' 라고 한다. 

나쁜 데이터베이스는 충분히 정규화되지 않은 것이다. 충분히 정규화된 데이터베이스는 더 이해하기 쉽고, 확장하기 쉬우며, insertion / deletion / update anomalies 로부터 보호된다. 

그렇다면 데이터베이스가 충분히 정규화 되었는지 어떻게 측정할 것인가? 이때 1NF, 2NF, 3NF .. 가 나온다. 숫자가 올라갈 수 록 기준이 더 엄격해진다. 마치 안전기준과 같다. 다리를 예시로 들어보자. 하나의 다리가 안전기준1 (사람이 건널 수 있음) 을 통과했다고 하자. 그렇다면 더 많은 기준을 적용해서 안전기준2 (자동차가 건널 수 있음) 가 만족하는지 테스트할 수 있다. 

## 제 1 정규형 (1NF)

친구가 Beatles 의 멤버가 누구였는지 물어봤다고 하자. 그렇다면 "John, Paul, George, Ringo" 라고 대답할 수 있다. 다른 친구는 "Paul, John, Ringo, George" 라고 대답할 수 있다. 이 2개의 대답은 다른 순서를 가지지만, 동일하다. 

데이터베이스에 이름들을 저장해보자. 
```SQL
SELECT Member_Name
FROM BEATLES;
```



| Beatle |
| ------ |
| John   |
| Paul   |
| Ringo  |
| George |

이것의 결과는  위의 표일 수 있다. 이때 순서는 상관없다. 

### 1NF 위반 (1) - row 의 순서로 의미를 전달할 때

하지만 만약에, 비틀즈의 키 순서대로 멤버들을 적었다고 하자. (Paul, John, George, Ringo) 이때는 순서가 의미가 생긴다. 하지만 이것은 정규화되지 않은 것이다. 왜냐하면 DB에서 row 의 순서는 아무런 의미가 없기 때문이다. 여기서 1NF 의 첫번째 위반이 생긴다. **row 의 순서를 이용해서 의미를 전달하는 것은 1NF 를 위반하는 것**이다. 

해결책은 어떻게 될까? 키라는 attribute 를 하나 더 만드는 것이다. 

| Beatle | Height |
| ------ | ------ |
| George | 178    |
| John   | 179    |
| Ringo  | 170    |
| Paul       |   180     |

### 1NF 위반(2) - 속성의 datatype 이 여러가지 일 때

| Beatle | Height |
| ------ | ------ |
| George | 178    |
| John   | 179    |
| Ringo  | 168과 171사이    |
| Paul       |   180     |

위의 표에는, Height 의 type이 Integer과 Strign 의 혼합된 형태이다. 이것은 1NF 를 위반한다. 속성의 datatype 는 1가지여야 한다. 

### 1NF 위반(3) - 테이블에 PK 가 없을 때

PK 는 테이블의 튜플을 특정할 수 있는 속성 혹은 속성의 집합이다. 위의 테이블에서, Beatles 의 멤버의 이름을 나타내는 Beatle 속성은 튜플 마다 유일하다. 따라서 Beatle 를 PK 로 정의하면 된다. 

```SQL
ALTER TABLE Beatles
ADD PRIMARY KEY (Beatle);
```

### 1NF 위반(4) - Repeating Groups 

| Player_ID | Inventory          |
| --------- | ------------------ |
| uhm       | 2 hammers, 4 rings |
| sso     | 18 coins           |
| eun          |  3 hammers, 4 arrows, 8 rings                  |

위의 테이블에서 Inventory 속성은 Repeating Group 이다. 각 Inventory 는 다른 그룹의 아이템들을 포함한다. Inventory 를 String 으로 저장하는 것은 최악의 경우인데, 왜냐면 query 를 날리는 것이 굉장히 어려워진다. 

| Player_ID | Quantity_1 | Item_Type_1 | Quantity_2 | Item_Type_2 | Quantity_3 | Item_Type_3 |
| --------- | ---------- | ----------- | ---------- | ----------- | ---------- | ----------- |
| uhm       | 2          | hammers     | 4          | rings       |            |             |
| sso          |  18          |  coins           |            |             |            |             |
|  eun         |  3          |   hammers          | 4           |  arrows           |  8          |  rings           |

위의 표에서는, Item 3개까지 수량과 유형을 기록할 수 있다. 하지만, 사용자는 100개 이상의 아이템을 가질 수 있어서 테이블의 크기가 너무 커진다. 

이렇게 반복되는 그룹의 아이템들을 하나의 row 에 저장하는 것은 1NF 에 위배된다.

| Player_ID | Item_Type | Item_Quantity |
| --------- | --------- | ------------- |
| uhm       | hammers   | 2             |
| uhm       | rings     | 4             |
| sso       | coins     | 18            |
| eun       | hammers          |  3             |
| eun          | arrows          |  4             |
| eun          | rings          |  8             |

위의 표는 1NF 를 만족한다. 이때 PK 는 (Player_ID, Item_Type) 의 집합이 된다. 



## 제 2 정규형(2NF)

위에 Player_Inventory 를 다시 보자. 이때 Player 마다 Rating이 있다고 하자.

| Player_ID | Item_Type | Item_Quantity |Player_Rating|
| --------- | --------- | ------------- |------|
| uhm       | hammers   | 2             |Intermediate|
| uhm       | rings     | 4             |Intermediate|
| sso       | coins     | 18            |Beginner|
| eun       | hammers          |  3             |Advanced|
| eun          | arrows          |  4             |Advanced|
| eun          | rings          |  8             |Advanced|

이것은 좋은 디자인이 아니다. 만약 sso 가 모든 코인을 잃어서 row 가 사라졌다고 하자. 그러면 sso 의 rating정보까지도 사라진다. 이것을 **<span style="background:#FEFBD1">deletion anomaly</span>** 라고 한다.

또, uhm 의 Rating 을 Intermediate 에서 Advanced 로 올리고 싶다고 하자. 테이블 내에 uhm 의 row 는 2개 있으므로 모두 업데이트 해야 한다. 하지만 오류가 생겨서 하나의 row 만 업데이트가 일어나고 나머지는 변함이 없다면, 논리적 오류가 생긴다. (하나의 사용자는 동시에 2개 레벨일 수 없다.) 이것을 **<span style="background:#FEFBD1">update anomaly</span>** 라고 한다. 

새로운 유저 cindy 가 게임에 참여했다고 하자. 이 사람은 Beginner 이다. cindy 는 Inventory 가 아무것도 없기 때문에, 위의 테이블에 저장할 수 없다. 이것을 **<span style="background:#FEFBD1">insertion anomaly</span>** 라고 한다. 

왜 이런 문제들이 생기는 걸까? 정답은 2NF 를 만족하지 않아서이다. 2NF 는 테이블의 non-key 속성들이 PK와 연관되어 있는 상태에 관한 것이다. 2NF 를 만족하기 위한 조건은, 각 non-key 속성은 PK 전체에 의존해야 한다. 

위의 테이블에서 non-key  속성은 Item_Quantity, Player_Rating 이다. PK는  (Player_ID, Item_Type)의 집합이다. Item_Quantity 는 (Player_ID, Item_Type) 전체에 의존한다. 이것을 나타내자면, $(Player\ ID, Item \ Type) \rightarrow \ Item\ Quantity$ 이다. 이것을 **<span style="background:#FEFBD1">Functional Dependency</span>** 라고 한다. 즉, 화살표 왼쪽에 있는 것과 화살표 오른쪽에 있는 것은 정확히 1:1 대응한다. 

하지만, Player_Rating 은 오로지 Player_ID 와 연관이 있다.  $Player \ Rating \rightarrow \ Player \ ID$ 이다. Player_ID 는 혼자서는 PK 가 아니라 PK 의 일부이기 때문에 문제가 생긴다. 

이 설계를 어떻게 고칠까?Player 정보와 Inventory 정보 테이이블을 분리하면 된다. 

| Player_ID | Player_Rating |
| --------- | ------------ |
| uhm       |  Intermediate            |
| sso       |  Beginner            |
| eun          | Advanced             |

여기서는 functional dependency 를 만족한다. 

## 제 3 정규형(3NF)

위의 테이블에서, Player Rating 옆에 구체적인 Level 정보를 추가하고 싶다고 하자. level 1~3 은 Beginner, 4~6 은 Intermediate, 7~9 는 Advanced 라고 하자. 

만약 Player 테이블에 Player_Skill_Level 속성이 추가되었다고 하자. 

| Player_ID | Player_Rating |Player_Level|
| --------- | ------------ |-----|
| uhm       |  Intermediate            |4|
| sso       |  Beginner            |3|
| eun          | Advanced             |8|

만약 sso 의 level 이 3에서 4로 증가했다고 하자. 그렇다면 Rating 도 Beginner 에서 Intermediate 로 바뀌어야 한다. 하지만 오류가 생겨서 Rating 이 업데이트가 안되면, 데이터간의 불일치가 생긴다. 

이것이 왜 생긴 문제일까 ? $Player \ Id \rightarrow \ Player \ Skill \ Level$ 이다.  그러나 Player Rating 은 Player Id 에 의존하지만, 간접적으로 의존한다. $Player \ Id \rightarrow \ Player \ Skill \ Level \rightarrow Player \{Rating}$  이다. 이것을 **<span style="background:#FEFBD1">Transitive Dependency</span>** 라고 한다. 

3NF 는 non-key 속성이 다른 non-key 속성에 의존하는 것을 허용하지 않는다. 따라서 위의 테이블을 3NF 를 위반한다. 

이것을 해결하는 방법은, Player 에는 (Player_ID, Player_Level) 속성을 두고, Player_Level 라는 새로운 테이블을 만들어 (Player_Level, Player_Rating) 을 저장하는 것이다. 

**3NF 의 핵심은, 모든 non-key 속성은 key, the whole key, nothing but the key 에만 의존해야 한다는 것**이다. 

## Boyce-Codd Normal Form(BCNF)

BCNF 는 제 3 정규형의 조금 더 강화된 형태이다. 예시를 보자.

| Sales_year | Sales_month | Widgets_Sold |
| ---------- | ----------- | ------------ |
| 2021       | November    | 1581         |
| 2021       | December    | 1927         |
| 2022           | January            | 1343             |

여기서 PK 는 (Sales_year, Sales_month) 이다. 이때 non-key 속성인 Widgets_Sold 는 key, the whole key, nothing but the whole key 에만 의존해야 한다는 제 3 정규형의 조건을 만족한다. 

| Locker_ID | Reservation_Start_Date | Reservation_End_Date | Reservation_End_Day |
| --------- | ---------------------- | -------------------- | ------------------- |
| 221       |  14-May-2019                      | 12-Jun-2019                     | Wednesday                    |
|  308         | 07-Jun-2019                       | 12-Jun-2019                     |  Wednesday                   |
| 537          |  14-May-2019                      |  17-May-2019                    |  Friday                   |

위의 테이블을 보자. (Locker_ID, Reservation_Start_Date) 를 PK 로 정의할 수 있다. 이때는 2NF 를 만족한다. 하지만 PK 를 (Locker_ID, Reservation_End_Date) 로도 정의할 수 있다. 만약 PK를 (Locker_ID, Reservation_End_Date) 로 정의한다면, Reservation_End_Day 는 PK 의 일부인 Reservation_End_Date 에만 의존하므로 2NF 를 위배한다. 

그렇다면 이것은 2NF 를 만족하는 것일까 아닐까? 정답은 '아니다' 이다. 2NF 의 정의를 좀 더 엄밀하게 보자. **2NF 는 non-prime 속성이 candidate key 의 일부에 의존하면 안된다.** 

**candidate key**(후보키)는 유일성과 최소성을 만족하는 속성 또는 속성들의 집합이다. 위의 테이블에는 (Locker_ID, Reservation_Start_Date) , Locker_ID, Reservation_End_Date) 2개의 후보키가 존재한다. **prime attribute** 는 최소 하나의 후보키에 속하는 속성이다. **non-prime attribute** 는 어떠한 후보키에도 속하지 않는 속성이다. 

위의 테이블에서 non-prime 속성은 Reservation_End_Day 뿐이다. 하지만 Reservation_End_Day 는 후보키의 일부인 Reservation_End_Date 에 의존하므로, 2NF 를 위배한다. 

3NF 의 엄밀한 정의도 보자. **각 non-prime 속성은 모든 후보키에 의존해야 하고, 후보키의 일부에 의존해서는 안되고, 다른 non-prime 속성에 의존해서는 안된다.** 

하지만 3NF 의 엄밀한 정의에는 약점이 있다. 아래 예시를 보자. 

| Release_year | Popularity_ranking | Movie_name      | Release_year_and_month |
| ------------ | ------------------ | --------------- | ---------------------- |
| 2008         | 1                  | The Dark Knight | 2008-07                |
| 2008         | 2                  | Indiana Jones   | 2008-05                |
| 2008         | 3                  | Kung Fu Panda   | 2008-06                |
| 2009         | 1                  | Avatar          | 2009-12                |
| 2009         | 2                  | Harry Potter    | 2009-07                | 
| 2009         | 3                  | Ice Age         |  2009-07                      |

3개의 후보키가 있다. (1) Movie_name, (2) (Release_year, Popularity_ranking), (3) (Release_year_and_month, Popularity_ranking) 이다. 여기서 주목할 점은 테이블 내의 모든 속성이 prime 속성이라는 것이다. 하지만 Release_year 는 Release_year_and_month 에 의존한다. 만약 Kung Fu Panda 의 Release_year_and_month 가 2018-06 으로 업데이트 되면, Release_year 과 데이터 불일치가 발생할 것이다. 

위 테이블은 3NF 를 만족한다. 왜냐하면 non-prime 속성이 없기 때문이다. 따라서 이 약점을 보완할만한 정규형인 **Boyce-Codd Normal Form(BCNF)** 가 도입되었다. 

BCNF 의 정의는 다음과 같다. **사소한 functinal dependencies 의 예외는 있을 수 있지만, 테이블 내에 모든 functinal dependency 는 후보키 또는 후보키의 super set 에 의존해야 한다.** 

사소한 functinal dependency 는 속성 그 자체 또는 속성 그 자체 + 다른 것 에 의존하는 것이다. 예를 들어 Popularity_ranking 이 Popularity_ranking 에 의존하거나, Popularity_ranking + (Popularity_ranking, Movie_name) 에 의존하는 것이다. 

후보키의 super set 을 **super key** 라고 한다. super key 는 유일성의 특성을 만족하는 속성 또는 속성들의 집합이다.

따라서 위의 테이블은 BCNF 를 만족하지 못한다. 왜냐하면 Release_year 는 Release_year_and_month 에 의존하는데, Release_year_and_month 는 super key 가 아니기 때문이다. 그렇다면 BCNF 를 만족하도록 어떻게 고칠까? Release_year_and_month 를 단순히 Release_month 로 바꾸면 된다. 그러면 더 이상 Release_year 가 Release_month 에 의존하지 않는다. 

BCNF 는 2NF, 3NF 의 정의를 엄밀하게 따져야 했다. 하지만 BCNF 를 informal 하게 나타내는 방법도 있다. 그것은 **테이블 내의 모든 속성이 key, the whole key, nothing but the key 에만 의존해야 한다는 것**이다. 여기서 key 는 후보키를 나타낸다. 

## 제 4 정규형(4NF)

제 3 정규형으로도 만족스럽지 않을 수 있다. 만약 BirdHouse 웹페이지를 디자인하고 싶다고 하자. 이때 Model, Color, Style 을 고를 수 있다. 선택지들을 테이블로 만들면 다음과 같다. 

<img width="210" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/a9c58b58-743c-4156-8a46-506f853bc90b">

| Model  | Color  | Style     |
| ------ | ------ | --------- |
| Tweety | Yellow | Bungalow  |
| Tweety | Yellow | Duplex    |
| Tweete | Blue   | Bungalow  |
| Tweety | Blue   | Duplex    |
| Metro  | Brown  | High-Rise |
| Metro  | Brown  | Modular   |
| Metro  | Grey   | High-Rise |
| Metro  | Grey   | Modular   |
| Priarie       |Brown       |Bungalow          |
| Priarie       |Brown       |Schoolhouse          |
| Priarie       |Beige       |Bungalow          |
| Priarie       |Beige       |Schoolhouse          |

이때 PK 는 (Model, Color, Style) 이다. Priarie 에 제공되는 color 는 Brown, Beige 였는데, 이때 새로운 color Green 이 생기면 새로운 열 2개 (Priarie, Green, Bungalow), (Priarie, Green, Schoolhouse) 를 추가해야 한다. 하지만 오류가 생겨서 (Priarie, Green, Schoolhouse) 열을 추가하지 못한다면 데이터 불일치가 발생한다. 

Model, Color, Style 과의 functional dependency 를 보자. 각 모델은 제공하는 color 의 집합이 있다. 이것을 **<span style="background:#FEFBD1">multi-valued dependency</span>** 라고 한다. 이것을 $Model \twoheadrightarrow Color$ 로 표현한다. 

**4NF 에서는, 테이블 내에 multivalued dependencies 는 key 에 대해서만 multivalued dependencies 이어야만 한다.** 여기서 Model 는 Key 가 아니기 때문에 4NF 를 만족하지 못한다. 

해결하는 방법은 2개의 테이블로 쪼개는 것이다. (Model, Color), (Model, Style) 테이블 2개로 쪼개면 된다. 

## 제 5 정규형(5NF)

아이스크림 가게 예시를 보자.

<img width="200" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/25ce9f44-47c8-440a-ab61-95f0d2cfdf8e">

친구 Jason 에게 어떤 아이스크림을 좋아하냐고 물었더니, Vanilla, Chocolate 를 좋아하고 Frosty, Alpine 브랜드를 좋아한다고 한다. 다른 친구 Susie 에게 물어보니, Rum Rasion, Mint Chocolate Chip, Strawberry 를 좋아하고 Alpine, Ice Queen 브랜드를 좋아한다고 한다. 이것을 테이블로 나타내 보자. 

| Person | Brand     | Flavor              |
| ------ | --------- | ------------------- |
| Jason  | Frosty's  | Vanilla             |
| Jason  | Frosty's  | Chocolate           |
| Jason  | Alpine    | Vanilla             |
| Susie  | Alpine    | Rum Rasion          |
| Susie  | Ice Queen | Mint Chocolate Chip |
| Susie       | Ice Queen          | Strawberry                    |

하지만 시간이 지나고, 취향이 바뀔 수 있다. Susie 는 이제 Frosty's 아이스크림도 좋아한다고 한다. 테이블을 업데이트 해야 한다. (Susie, Frosty's, Strawberry) 와 
 (Susie, Frosty's, Mint Chocolate Chip )  을 추가해야 하지만, 추가하는 과정에서 오류가 생길 수 있다.  (Susie, Frosty's, Mint Chocolate Chip ) 를 추가 하는 것을 실패했다면 데이터의 불일치가 생긴다. 

우리는 3가지의 정보가 주어졌다. (1) 어떤 사람이 어떤 브랜드를 좋아하는지, (2) 어떤 사람이 어떤 맛을 좋아하는지, (3) 어떤 맛이 어떤 브랜드에 있는지. 이 3가지 정보를 바탕으로 3개의 테이블을 만들면 된다. (Brand, Flavor) , (Person, Brand), (Person, Flavor). 이렇게 모든 정보를 나타낼 수 있다.

**5NF 란 하나의 테이블이 다른 테이블을의 join 으로 인해 만들어질 수 없는 것이다.** 

## Review 

각 정규화 레벨이 커질 수록, 아래 레벨의 정규화는 모두 만족하는 것이다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**1NF**</span></span>  
- 튜플의 순서를 이용해 의미를 전달하는 것은 허용되지 않는다. 
- 같은 칼럼내에 다른 datatype 를 같이 쓰는 것은 허용되지 않는다.
- PK 가 없는 테이블은 허용되지 않는다.
- Repeating Groups 는 허용되지 않는다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**2NF**</span></span>  
- 테이블 내에 각 non-key 속성은 PK 전체에 의존해야 한다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**3NF**</span></span>  
- 테이블 내에 모든 non-key 속성은 key, the whole key, nothing but the key 에만 의존해야 한다
 
여기서 **<span style="background:#FEFBD1">BCNF (Boyce-Codd Normal Form)</span>** 이 나오는데, BCNF 는 테이블 내에 모든 속성은 key, the whole key, nothing but the key 에만 의존해야 한다. 

<span style="font-size:110%"><span style="background-color: #EBFFDA">**4NF**</span></span>  
- 테이블 내에 multivalued dependencies 는 key 에 대해서만 multivalued dependencies 이어야만 한다.

<span style="font-size:110%"><span style="background-color: #EBFFDA">**5NF**</span></span>  
- 하나의 테이블이 다른 테이블을의 join 으로 인해 만들어질 수 없어야 한다. 


## Reference
- [Learn Database Normalization - 1NF, 2NF, 3NF, 4NF, 5NF](https://www.youtube.com/watch?v=GFQaEYEc8_8)
