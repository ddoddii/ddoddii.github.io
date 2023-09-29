+++
author = "Soeun"
title = "[Spring] IoC & DI & Spring Bean"
date = "2023-09-28"
description = "Spring 도입 이유 - Java 클린 코드에서부터"
categories = [
    "CS"
]
tags = [
    "Spring"
]
image = ""
+++


## Inversion of Control (IoC) 

> IoC 는 의존 관계 주입(Dependency Injection) 이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는 것이 아니라, 주입 받아 사용하는 것이다.


일반적인 의존성에 대한 제어권은 보통 개발자가 가지고 있다. 일반적인 흐름의 코드(개발자가 제어권을 가지고 있는 코드) 에 대해 먼저 살펴보자.

```java
import core.discount.DiscountPolicy;

public final class SellBookService {  
	private String title;  
    private Set<String> authors;
	private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    
  
	public SellBookService(String title, Set<String> authors, DiscountPolicy discountPolicy) {  
        this.title = title;  
        this.authors = authors;  
        this.discountPolicy = discountPolicy;
    }
```

SellBookService 에서 , discountPolicy 라는 필드를 가지고 있다. 그리고 constructor 내부에서 직접 생성해서 필드를 초기화 하고 있다. (this.** = ** 를 통해서) 즉, 객체 생명주기나 메서의 호출을 개발자가 직접 제어하고 있다. 생명주기라는 말이 너무 거창한데, 그냥 객체가 언제 생겨나고 없어지는지를 개발자가 직접 제어할 권한이 있는 것이다.  자세하게는, `객체 생성 -> 의존 설정 -> 초기화 -> 사용 -> 소멸` 의 주기이다. 

SellBookService Constructor 를 보면, 새로운 인스턴스를 만들 때마다, `new FixDiscountPolicy` 를 통해 새로운 BuyBookService Object를 만든다. 근데 실제 현업에서는 클래스가 1000개가 넘는다. 이때마다 새로운 Object가 생성되면 문제이다. 

그리고 만약에 FixDiscountPolicy 가 아니라, RateDiscountPolicy 로 변경하고 싶으면, 이 실행코드 내부에서 FixDiscountPolicy -> RateDiscountPolicy 로 바꾸어 주어야 한다. 

이 코드는 OCP 위반이다. 지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다. 그리고, SellBookService 는 DiscountPolicy 인터페이스 뿐만 아니라, FixDiscountPolicy 라는 구체 클래스에도 의존하고 있다. 따라서 DIP 위반이다. 

이러한 문제점들을 IoC (제어권 역전)을 통해서 해결할 수 있다. IoC 를 구현하는 패턴들에는 여러가지가 있다. 
- Service Locator
- Factory
- Abstract Factory
- Strategy
- Template Method
- Dependency Injection

여기서는 DI 에 대해서 우선 알아보고자 한다. 그 전에 IoC 개념을 이용해 구분하는 프레임워크와 라이브러리의 차이를 알아보자.

### 프레임워크 vs. 라이브러리
- 프레임워크 : 내가 작성한 코드를 제어하고, 대신 실행하는 것 -> IoC 개념이 들어감. 
- 라이브러리 : 내가 작성한 코드가 직접 제어의 흐름을 담당하는 것 

## Dependency Injection(DI)

위의 문제점들을 해결하기 위해선, 구체 클래스에 의존하는 것이 아니라 인터페이스에만 의존하도록 코드 구조를 변경해야 한다. 여기서 **의존성 주입**이라는 개념이 나타난다. 

```java
import core.discount.DiscountPolicy;

public final class SellBookService {  
	private String title;  
    private Set<String> authors;
	private final DiscountPolicy discountPolicy;
  
    /**  
     * Builds a book with the given title and authors.     *     * @param title the title of the book  
     * @param authors the authors of the book  
     */    
     public BuyBookService(String title, Set<String> authors, DiscountPolicy discountPolicy) {  
        this.title = title;  
        this.authors = authors;  
        this.discountPolicy = discountPolicy;
    }
```

위에처럼 `new FixDiscountPolicy` 를 없앤다면 ? 그러면 더 이상 구체 클래스에 의존하지 않고, 인터페이스에만 의존한다. 그렇지만 BuyBookService 입장에서는 구체적으로 discountPolicy 에 어떤 구체적인 객체가 들어올지 모른다. 그래서 구체적인 어떤 객체를 주입할지는 외부에서 결정해준다. 이것이 **Dependency Injection** 이다. 

실행코드인 SellBookService 는 오직 '책을 파는 행위' 에만 집중하고, 구체적으로 어떠한 할인 정책을 쓸 것인지는 **외부에서 결정해서 알려주는 것**이다 ! 

외부 세력으로는, java 코드로만 작성하자면 우선 AppConfig 파일을 작성할 수 있다. AppConfig 파일을 통해, 실제 구현 객체를 생성하고, 실행하는 파일과 연결할 수 있다. AppConfig 처럼 객체를 생성하고 의존관계를 연결해주는 것을 **IoC 컨테이너** 혹은 **DI 컨테이너**라고 한다. 

```Java
import core.discount.DiscountPolicy;
import core.discount.RateDiscountPolicy;
import core.book.BuyBookService;

public class AppConfig{

	private String title;
	private Set<String> authors;


	public BuyBookService buybookService(){
		return new  buybookService(
			title, 
			authors, 
			new RateDiscountPolicy()
		);
	}
	
}
```

실제 실행 코드 - BookApp

```java
public class BookApp {
	public static void main(String[] args) {
         AppConfig appConfig = new AppConfig();
         BuyBookService buybookService = appConfig.buybookService();

}
```

이렇게 하면, 실제 실행 될때 AppConfig 파일에서 구체 클래스를 받은 뒤, buybookService 를 실행한다. 

이렇게 위에 

(간소화한 코드라서 오류가 뜰 수 있다.)


### 의존 관계

의존 관계에도 2가지 유형이 있다. 정적인 클래스 의존 관계와, 실행 시점에 결정되는 동적인 객체의 의존관계이다. 

#### 정적인 클래스 의존 관계

클래스가 사용하는 import 코드만 보고 알 수 있다. SellBookService 는 DiscountPolicy 에 의존한다. 하지만 이 의존관계만 가지고는, 실제 어떠한 구체적인 할인정책 (DiscountPolicy 의 구체적인 구현 클래스) 가 주입될 지 모른다. 

#### 동적인 인스턴스 의존 관계

어플리케이션 실행 시점에서 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계이다. 
어플리케이션 실행 시점에서 외부에서 실제 구현 객체를 생성하고, 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결되는 것을 **의존관계 주입**이라고 한다. 

## Spring 의 도입

위의 코드들은 모두 순수한 Java 로 작성된 코드이다. IoC, DI 도 자바로만 할 수 있는데, 왜 굳이 Spring 을 도입해서 써야 할까 ? 왜냐하면, @Configuration, @Bean 과 같은 어노테이션을 사용함으로써 얻을 수 있는 이득이 많기 때문이다 ! 

앞서 작성했던 AppConfig 파일에 Spring 을 도입해보자.

```Java
import core.discount.DiscountPolicy;
import core.discount.RateDiscountPolicy;
import core.book.BuyBookService;

@Configuration
public class AppConfig{

	private String title;
	private Set<String> authors;

	@Bean
	public BuyBookService buybookService(){
		return new  buybookService(
			title, 
			authors, 
			new RateDiscountPolicy()
		);
	}
}
```

class 위에는 @Configuration, 메서드 위에는 @Bean 를 붙여주었다. 이렇게 하면 스프링에서 자동으로 스프링 컨테이너에 스프링 빈으로 등록해준다. 

### Bean 

> 스프링 빈은 다른 메타데이터와 함께 스프링 컨테이너에 의해 관리되는 object 이다. 

스프링 컨테이너는 여러 개의 스프링 빈을 관리한다. 빈을 접근하기 위해서는, 아래 처럼 `getBean()` 메서드를 사용할 수 있다. 
 
```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);  
  
BuyBookService buyBookService = ac.getBean("buybookService", BuyBookService.class);
```

Bean 에 저장되는 정보에는 메타데이터도 있다. 여기서 빈에 의해 관리되는 metadata 는 아래와 같다.

- BeanClassName : 생성할 빈의 클래스 이름
- factoryBeanName : 팩토리 역할의 빈을 사용할 경우의 이름 (appConfig, … 등)
- factoryMethodName : 빈을 생성할 팩토리 메서드 지정 (memberService, … 등)
- Scope : Singleton(기본값) or Property
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연 처리하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음

빈을 조회할 때, 부모 타입으로 조회하면, 부모와 자식 타입이 모두 조회된다. 

```java
void findAllBeanByParentType() {
         Map<String, DiscountPolicy> beansOfType =
 ac.getBeansOfType(DiscountPolicy.class);
```

이렇게 DiscountPolicy 를 조회하면, DiscountPolicy 의 인터페이스를 구현한 FixDiscountPolicy 와 RateDiscountPolicy 를 모두 조회할 수 있다. 

## Reference
- Spring 핵심 원리
- https://www.youtube.com/watch?v=oqYRl06DNHQ
- https://youtu.be/L-0UvbFUXrk?si=wf0lP6Q6ZqpRysfT