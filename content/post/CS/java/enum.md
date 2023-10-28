+++
author = "Soeun"
title = "[Java] Enum"
date = "2023-10-24"
description = "관련된 상수의 집합인 Enum"
categories = [
    "CS"
]
tags = [
    "Java"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/384bd77d-965b-4d8d-aa98-015cedc6ad55"
+++

## Enum

Java Enum 이란 <span style="background:#fff88f">관련된 상수들을 같이 묶어놓은 것</span>이다. Java 는 타입에 안전한 열거형을 제공한다. 즉 비교할 때 **값 , 타입 둘 다 체크**한다. 

첫번째 예시로 야구게임을 보자. 야구 게임에서, 결과가 Strike, Ball, Nothing 으로 나온다고 하자. 이때 Enum 클래스로 상수값을 관리할 수 있다. Enum 도 클래스이므로, 참조 객체형으로 쓸 수 있다. 

```java
public enum Result{
	BALL, STRIKE, NOTHING;
}

class Baseball{
	final Result result;
	
	if (isStrike()){
		return result.STRIKE;
	}
}
```

두번째 예시로, 카드에 관한 클래스를 보자.
```java
class Card {
	static final int CLOVER = 0;
	static final int HEART = 1;
	static final int DIAMOND = 2;
	static final int SPADE = 3;

	static final int ZERO = 0;
	static final int ONE = 1;
	static final int TWO = 2;

}
```

이것을 단순하게 enum 으로 바꿔서 나타낼 수 있다.
```java
class Card{
	enum Kind {CLOVER, HEARD, DIAMOND, SPADE}
	enum Value {ZERO, ONE, TWO}

	final Kind kind;
	final Value value;
}
```

여기서 Card.CLOVER 과 Card.ZERO 는 값이 0 으로 같다. 하지만 Java enum 에서는 값 & 타입 모두 비교하기 때문에, 애초이 `Card.Kind.CLOVER == Card.VALUE.ZERO`를 하면 타입이 달라서 컴파일 에러가 난다. 

## Enum 정의와 사용
### Enum 정의
```java
enum Direction {EAST, SOUTH, WEST, NORTH}
```
위와 같이 정의하면, EAST, SOUTH, WEST, NORTH 각각에 0,1,2,3 정수값이 부여된다. 

### Enum 사용
```java
class Unit{
	int x,y;
	Direction dir;
	void init(){
		dir = Direction.EAST;
	}

	void travel(){
		if (dir == Direction.EAST){
			x++;
		}
		else if(dir.compareTo(Direction.WEST) > 0){
			y++;
		}
	}
}
```
`dir` 변수에는 EAST, SOUTH, WEST, NORTH 값 중 하나만 들어올 수 있다. 열거형 상수에 == 와 `compareTo()` 사용 가능하다. `compareTo()` 는 같으면 0, 왼쪽이 크면 양수, 오른쪽이 크면 음수를 리턴한다. 비교연산자(>) 는 사용 불가능하다. 

### Enum 조상
| 메서드                                    | 설명                                                |
| ----------------------------------------- | --------------------------------------------------- |
| String name()                             | 열거형 상수의 이름을 문자열로 반환                  |
| int ordinal()                             | 열거형 상수가 정의된 순서를 반환(0부터)  -> 상수가 가진 실제 값과 상관 없음 !!           |
| T valueOf(Class&lt;T&gt; enumType, String name) | 지정된 열거형에서 name 과 일치하는 열거형 상수 반환 |
| Class&lt;E&gt; getDeclaringClass()                 | 열거형의 Class 객체 반환                            |

`values()` , `valuesOf()` 는 컴파일러가 추가해준다. 
- static E[] values
- static E valueOf(String name)

```java
void printDir(){   
    Direction d = Direction.valueOf("WEST");  
    System.out.println(d.name());  // WEST
    System.out.println(d.ordinal()); // 2
}
```
 
### Enum 에 멤버 추가하기
원하는 값을 괄호() 안에 적는다. 값이 여러개 일 수 도 있다. 값의 개수만큼 필드를 추가해주면 된다. 

```java
enum Direction {
	EAST("동",">"), WEST("서","<"),SOUTH("남","V"),NORTH("북","^");
	private String dir;
	private String symbol; //값을 저장할 필드 추가
	Direction(String dir, String symbol){ // 생성자 추가
		this.dir = dir;
		this.symbol = symbol;
	}
	public String getDir() {return dir;}
	public String getSymbol() {return symbol;}
}
```
Enum 의 생성자는 private 이므로, 외부에서 객체 생성 불가하다. 
```java
Direction d = new Direction("남동"); // 에러 !! 열거형의 생성자는 외부에서 호출 불가
```


## Enum 은 언제 활용할까?

Enum 의 장점은 아래와 같다고 한다. (from 우아한기술블로그)
- 문자열과 비교해, **IDE의 적극적인 지원**을 받을 수 있습니다.
    - 자동완성, 오타검증, 텍스트 리팩토링 등등
- 허용 가능한 값들을 제한할 수 있습니다.
- **리팩토링시 변경 범위가 최소화** 됩니다.
    - 내용의 추가가 필요하더라도, Enum 코드외에 수정할 필요가 없습니다.

 만약 같은 의미를 나타내는 값이 여러 개 있다고 하자.  Y/N , 1/0 , true/false 등이 각각 같은 의미로 사용될 수 있다. 이것을 Enum 으로 묶어버리는 것이다. 
 ```java
 public enum Status{
	 Y("1",true),
	 N("0",false)

	//생성자 추가...
 }
```

이것 말고도 같은 의미를 가진 데이터를 같이 관리하는 것에 대한 장점이 많다. 앞으로 '관련된 상수' 는 Enum 도입을 적극적으로 생각해봐야겠다 !! 


##  Referecne
- https://www.youtube.com/watch?v=ODHC-n4mpMY
- https://www.youtube.com/watch?v=R0WrMaKoLTE
- https://techblog.woowahan.com/2527/