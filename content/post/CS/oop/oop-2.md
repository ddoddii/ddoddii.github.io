+++
author = "Soeun"
title = "객체의 역할, 책임, 협력"
date = "2023-10-26"
summary = "객체는 어떻게 협력하는가?"
categories = [
    "CS"
]
tags = [
    "oop"
]
+++

## 협력
객체끼리는 서로 협력한다. 협력은 한 객체가 다른 객체에게 요청을 보낼 때 시작된다. 이상한 나라의 앨리스에서 열린 재판으로 협력 관계를 살펴보자.

누군가 왕에게 재판을 요청한다. 왕이 하얀 토끼에게 증인을 부를 것을 요청한다. 하얀 토끼는 모자 장수에게 증인석으로 입장할 것을 요청한다. 모자 장수는 토끼의 요청에 응답해서 증인석에 입장한다. 이제 왕은 모자장수에게 증언할 것을 요청한다. 모자 장수는 증언함으로써 응답한다. 

코드로 살펴보자.
```java
public class King {  
    public void openCourt(Rabbit rabbit, Witness witness){  
        rabbit.callWitness(witness);  
        witness.sayInCourt();  
    }  
}

public class Rabbit {  
    void callWitness(HatDealer hatdealer){  
        System.out.println("Calling Witenss");  
        witness.stepInCourt();  
        System.out.println("Called Witness");  
    }  
}

public class HatDealer {  
    String name;  
    public HatDealer(String name){  
        this.name = name;  
    }
    void stepInCourt(){  
    System.out.println(this.name + " in court");  
	}  
  
	void sayInCourt(){  
	    System.out.println(this.name + " saw this, your majesty");  
	}

public class World(){
	TrumpKing king = new TrumpKing();  
	Rabbit rabbit = new Rabbit();  
	HatDealer dealer = new HatDealer("James");
	
	void judgingInCourt(){  
	    king.openCourt(rabbit, witness);  
	}  
	  
	public static void main(String[] args) {  
	    World strangeWorld = new World();  
	    strangeWorld.judgingInCourt();  
	}
}

```

위의 코드를 실행하면 어떻게 될까? (java 이므로 클래스마다 다른 파일에 있고, World 클래스를 실행하면 된다.) 
```text
Calling Witenss
James in court
Called Witness
James saw this, your majesty
```
위의 결과가 나올 것이다. 

위의 스토리에서 있었던 요청, 응답을 그대로 코드로 구현한 것이다. King, HatDealer, Rabbit 은 각각 서로에게 메세지를 보냄으로써 협력한다. 

## 책임

어떤 대상에 대한 요청은 그 대상이 요청을 처리할 책임이 있음을 암시한다. 객체의 책임은 '객체가 무엇을 알고 있는지(knowing)' 과 '무엇을 할 수 있는지(doing)' 로 구성된다. 
- 하는 것
	- 객체를 생성하거나 계산을 하는 등 스스로 하는 것
	- 다른 객체의 행동을 시작시키는 것
	- 다른 객체의 활동을 제어하고 조절하는 것
- 아는 것
	- 개인적인 정보에 관해 아는 것
	- 관련된 객체에 대해 아는 것
	- 자신이 유도하거나 계산할 수 있는 것에 관해 아는 것


객체지향에서 가장 중요한 것은 <span style="background:#fff88f">적절한 객체에게 적절한 책임을 할당</span>하는 것이다. 

다른 객체에게 책임을 수행하도록 요청하는 것을 <span style="background:#fff88f">**메세지 전송**</span>이라고 한다. 

## 역할

협력의 관점에서, 어떤 객체가 어떤 책임을 수행하는 것이 어떤 의미인지 보자. 위의 예시에서 모자장수가 재판에서 증언을 했다. 이것은 모자장수가 '증언'의 역할을 한 것이고, 왕은 '판사' 라는 역할을 수행한 것이다. 

역할은 재사용 가능하고 유연한 객체지향 설계를 가능하게 한다. 만약 위와 같은 상황에서 '판사' 역할을 여왕이 할 수 도 있고, '증언'의 역할은 앨리스가 할 수 있다. 그럴때마다 위의 코드를 다시 짜려면 너무 낭비일 것이다. 따라서 하나의 협력으로 다루고, 역할을 도입하는 것이다. 해당 역할을 수행할 수 있는 객체로 언제든지 대체할 수 있다. 역할을 수행할 수 있다는 의미는, 동일한 메세지를 이해할 수 있는 객체라는 의미이다. 

역할을 도입해서, <span style="background:#fff88f">협력을 추상화</span>할 수 있다. 

## 객체지향 설계 기법
역할, 책임, 협력의 관점에서 어플리케이션을 설계하는 방법에 대해 알아보자. 각 설계 기법에 대해서는 다른 포스팅에서 더 자세히 다룰 예정이다. 

- Responsibility-Driven Design
	- 협력에 필요한 객체들을 식별하고, 적절한 객체에게 책임을 부여한는 방식이다. 
- Design Pattern
	- 반복적으로 나오는 해결방법을 정의한 설계 템플릿이다.
- Test-Driven Development
	- 테스트를 먼저 작성하고 테스트를 통과하는 구체적인 코드를 추가하면서 어플리케이션을 완성하는 방식이다. 



## Reference
- 객체지향의 사실과 오해, ch.4



