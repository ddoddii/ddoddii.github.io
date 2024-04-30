+++
author = "Soeun"
title = "스프링에서 의존관계를 설정하는 방법은 무엇인가?"
date = "2024-03-24"
summary = "Spring Core 2편 - Dependencies"
categories = [
    "CS"
]
tags = [
    "Spring"
]
slug = "dependencies"
series = ["Spring Core"]
series_order = 2
+++

{{< alert icon="edit">}}
본 포스팅은 **Spring Core 공식 문서**를 번역하고, 추가적으로 제 생각을 덧붙인 글입니다.
{{< /alert >}}

## Dependency Injection

의존성 주입(DI) 는 오브젝트들이 그들의 의존성을 오로지 생성자 인자, Setter 기반, 팩토리 메서드의 인자를 통해 정의하는 방법입니다. 

스프링 컨테이너는 이 의존성들을 빈을 생성할 때 주입해줍니다. 이 프로세스는 빈이 자신의 의존성을 판단하고 본인을 생성하는 단계와 정확히 반대의 순서이기 때문에 제어의 역전이라고 불립니다. 

DI 원칙을 사용하면, 코드는 훨씬 깨끗하고, 오브젝트가 의존성과 같이 제공되면 분리하는 것이 훨씬 쉽습니다. 오브젝트는 직접 의존성들을 찾지 않고 의존성들의 위치 또는 클래스들에 대해 모릅니다.  결과적으로 인터페이스에 의존하게 되면, 유닛 테스트를 위해 mock 을 하는 것이 가능해져 클래스들은 훨씬 테스트하기 쉬워집니다. 

### Constructor-based Dependency Injection

생성자 기반 DI는 컨테이너가 몇개의 인자들을 가진 생성자를 호출할 때 이루어집니다. 

```java
public class SimpleMovieLister {

	// the SimpleMovieLister has a dependency on a MovieFinder
	private final MovieFinder movieFinder;

	// a constructor so that the Spring container can inject a MovieFinder
	public SimpleMovieLister(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// business logic that actually uses the injected MovieFinder is omitted...
}
```

#### Constructor Argument Resolution

생성자 인자 결정은 인자의 타입을 사용합니다. 빈 정의에서 생성자 인자의 모호성이 전혀 없으면, 빈에 생성자의 인자들이 정의된 순서가 빈의 인스턴스화 될 때 생성자에게 공급되는 인자의 순서입니다. 

```java
package x.y;

public class ThingOne {

	public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
		// ...
	}
}
```

`ThingTwo` 와 `ThingThree` 사이에는 어떠한 관계(상속)도 없으면, 모호성은 없습니다. 따라서 아래의 설정은 잘 동작하고,  `<constructor-arg/>` 요소에 생성자 인자 인덱스 또는 타입을 지정하지 않아도 됩니다. 

```xml
<beans>
	<bean id="beanOne" class="x.y.ThingOne">
		<constructor-arg ref="beanTwo"/>
		<constructor-arg ref="beanThree"/>
	</bean>

	<bean id="beanTwo" class="x.y.ThingTwo"/>

	<bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

만약 간단한 타입이 사용되는 경우 (`<value>true</value>`) 스프링은 값의 타입을 결정할 수 없고, 도움 없이 타입을 매칭할 수 없습니다. 

```java
package examples;

public class ExampleBean {

	// Number of years to calculate the Ultimate Answer
	private final int years;

	// The Answer to Life, the Universe, and Everything
	private final String ultimateAnswer;

	public ExampleBean(int years, String ultimateAnswer) {
		this.years = years;
		this.ultimateAnswer = ultimateAnswer;
	}
}
```

여기서는, 생성자의 인자를 `type` 속성을 이용해 명확하게 명시하면, 컨테이너는 타입 매칭을 사용할 수 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg type="int" value="7500000"/>
	<constructor-arg type="java.lang.String" value="42"/>
</bean>
```

또한 `index` 속성을 사용해서 생성자의 인덱스를 지정할 수 도 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg index="0" value="7500000"/>
	<constructor-arg index="1" value="42"/>
</bean>
```

또한 생성자의 파라미터 이름을 사용할 수도 있습니다.

```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg name="years" value="7500000"/>
	<constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

생성자의 파라미터 이름을 사용하려면, 스프링이 알아볼 수 있게 플래크를 명시해야 합니다. 이것은 `@ConstructorProperties` 어노테이션을 사용해서 할 수 있습니다. 

```java
package examples;

public class ExampleBean {

	// Fields omitted

	@ConstructorProperties({"years", "ultimateAnswer"})
	public ExampleBean(int years, String ultimateAnswer) {
		this.years = years;
		this.ultimateAnswer = ultimateAnswer;
	}
}
```

### Setter-based Dependency Injection

Setter 기반 DI는 no-argument 생성자 또는 no-argument static factory method 를 호출한 후, 컨테이너가 빈에 있는 setter 메서드를 호출하면서 이루어집니다. 

```java
public class SimpleMovieLister {

	// the SimpleMovieLister has a dependency on the MovieFinder
	private MovieFinder movieFinder;

	// a setter method so that the Spring container can inject a MovieFinder
	public void setMovieFinder(MovieFinder movieFinder) {
		this.movieFinder = movieFinder;
	}

	// business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext` 는 관리하는 빈에 대해 생성자 기반과 Setter 기반 DI를 제공합니다. 생성자로 인해서 의존성이 주입된 후에도, setter 기반 의존성 주입을 할 수 있습니다. 의존성들을 `BeanDefinition` 형태로 설정할 수 있고, 프로퍼티들을 한 형태에서 다른 형태로 바꾸기 위해 `PropertyEditor` 인스턴스들과 함께 사용할 수 있습니다. 그렇지만 대부분의 스프링 유저들은 클래스를 직접적으로 이용하는 대신 xml 기반 설정정보, 어노테이션(@Component, @Controller ...), @Configuration 클래스에 있는 @Bean 메서드 들을 사용합니다. 이러한 소스들은 내부적으로 BeanDefition 인스턴스로 변환되고, 전체 스프링 IoC 컨테이너 인스턴스를 로드하기 위해 사용됩니다. 

> Constructor-based or setter-based DI?

생성자 기반 DI와 세터 기반 DI를 둘 다 사용할 수 있으므로,  필수적인 의존성들은 생성자 기반 DI를 사용하고, 부가적인 의존성들은 세터 기반 DI를 사용하는 것이 가장 좋은 방법입니다. setter 메서드에 @Autowired 어노테이션을 사용하면 해당 프로퍼티를 필수 의존성으로 만들 수 있지만, 프로그래밍 방식의 인자 검증과 함께 생성자 주입하는 방식이 더 낫습니다. 

스프링 팀에서는 일반적으로 생성자 주입을 지지합니다. 이를 통해 애플리케이션 컴포넌트를 불변 객체로 구현할 수 있고, 필수 의존성이 'null'이 아님을 보장할 수 있기 때문입니다. 또한 생성자로 주입된 컴포넌트는 항상 완전히 초기화된 상태로 클라이언트(호출) 코드에 반환됩니다. 참고로, 많은 수의 생성자 인자는 좋지 않은 코드의 징후이며, 해당 클래스가 너무 많은 책임을 가지고 있어 관심사의 적절한 분리를 더 잘 처리하기 위해 리팩토링해야 함을 의미합니다.

Setter 주입은 주로 클래스 내에서 합리적인 기본값을 할당할 수 있는 선택적 의존성에만 사용해야 합니다. 그렇지 않으면 코드에서 의존성을 사용하는 모든 곳에서 not-null 검사를 수행해야 합니다. Setter 주입의 한 가지 이점은 setter 메서드가 해당 클래스의 객체를 나중에 재구성하거나 다시 주입할 수 있게 만든다는 것입니다. 따라서 JMX MBean을 통한 관리는 setter 주입의 매력적인 사용 사례입니다.

특정 클래스에 가장 적합한 DI 스타일을 사용하세요. 때로는 소스 코드가 없는 제3자 클래스를 다룰 때 선택의 여지가 없습니다. 예를 들어, 제3자 클래스가 setter 메서드를 노출하지 않는 경우 생성자 주입이 유일한 DI 형태일 수 있습니다.

### Dependency Resolution Process

컨테이너는 빈 의존성을 아래의 방식으로 해결합니다. 
- `ApplicationContext`는 모든 빈을 설명하는 구성 메타데이터로 생성되고 초기화됩니다. 구성 메타데이터는 XML, Java 코드 또는 어노테이션으로 지정할 수 있습니다.
- 각 빈에 대해 의존성은 속성, 생성자 인자 또는 정적 팩토리 메서드(일반 생성자 대신 사용하는 경우)에 대한 인자의 형태로 표현됩니다. 이러한 의존성은 빈이 실제로 생성될 때 빈에 제공됩니다.
- 각 속성이나 생성자 인수는 설정할 값의 실제 정의이거나 컨테이너의 다른 빈에 대한 참조입니다.
- 값인 각 속성이나 생성자 인수는 지정된 형식에서 해당 속성이나 생성자 인수의 실제 타입으로 변환됩니다. 기본적으로 Spring은 문자열 형식으로 제공된 값을 `int`, `long`, `String`, `boolean` 등과 같은 모든 내장 타입으로 변환할 수 있습니다.

Spring 컨테이너는 컨테이너가 생성될 때 각 빈의 구성을 검증합니다. 그러나 빈 속성 자체는 빈이 실제로 생성될 때까지 설정되지 않습니다. 싱글톤 스코프이고 사전 인스턴스화(기본값)로 설정된 빈은 컨테이너가 생성될 때 생성됩니다. 스코프는 빈 스코프에 정의되어 있습니다. 그렇지 않으면 빈은 요청될 때만 생성됩니다. 빈의 생성은 잠재적으로 빈의 의존성과 의존성의 의존성(등등)이 생성되고 할당됨에 따라 빈의 그래프가 생성될 수 있습니다. 이러한 의존성 간의 해결 불일치는 나중에(즉, 영향을 받는 빈이 처음 생성될 때) 나타날 수 있습니다.

> **Circular Dependencies**
> 
> 주로 생성자 주입을 사용하는 경우, 해결 불가능한 순환 의존성 시나리오가 발생할 수 있습니다. 
> 
> 예를 들어, A 클래스가 생성자 주입을 통해 B 클래스의 인스턴스를 필요로 하고, B 클래스는 생성자 주입을 통해 A 클래스의 인스턴스를 필요로 하는 경우입니다. A와 B 클래스의 빈이 서로 주입되도록 구성하면, Spring IoC 컨테이너는 런타임에 이 순환 참조를 감지하고 `BeanCurrentlyInCreationException`을 던집니다. 
> 
> 한 가지 가능한 해결책은 일부 클래스의 소스 코드를 수정하여 생성자가 아닌 setter로 구성하는 것입니다. 또는 생성자 주입을 피하고 setter 주입만 사용하는 방법도 있습니다. 즉, 권장되지는 않지만 setter 주입으로 순환 의존성을 구성할 수 있습니다.
> 
> 일반적인 경우(순환 의존성이 없는 경우)와 달리, 빈 A와 빈 B 사이의 순환 의존성은 한 빈이 완전히 초기화되기 전에 다른 빈에 주입되도록 강제합니다(고전적인 닭이 먼저냐 달걀이 먼저냐 시나리오).

스프링은 설정정보에서 문제(존재하지 않는 빈 참조 혹은 순환 의존성)를 컨테이너 로드 시점에 감지합니다. 스프링은 이러한 문제에 대한 해결을 빈이 실제로 생성될 시점까지, 최대한 미룹니다. 이것은 처음에 오류 없이 로드한 스프링 컨테이너가 사용자가 오브젝트를 요청할 시 예외를 발생시킬 수 있다는 것을 뜻합니다. 예를 들어, 빈이 없거나 유효하지 않은 프로퍼티에 대한 예외를 던질 수 있습니다. 이러한 설정정보에 이슈에 대한늦은 발견은, `ApplicationContext` 가 사전 인스턴스화된 싱글턴 빈들을 구현하는 이유입니다. 처음에 약간의 메모리와 시간을 소비해서, 실제 필요한 시점보다 일찍 빈들을 미리 만들어 놓으면, 실제 빈을 요청하는 시점이 아닌 `ApplicationContext` 가 생성될 당시에 설정정보에 대한 문제점을 알 수 있습니다. 

만약 초기 설정 때 빈들을 만들고 싶지 않으면, 설정정보에 `lazy-init` 속성을 추가해서 **lazy initializalition** 을 할 수도 있습니다. 

아래는 xml 설정정보의 예시입니다.

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

순환 의존성이 없는 경우, 하나 이상의 협력하는 빈이 의존하는 빈에 주입될 때 각 협력 빈은 의존 빈에 주입되기 전에 완전히 구성됩니다. 즉, 빈A 가 빈B 에 의존하는 경우, 스프링 IOC 컨테이너는 빈A의 setter 메서드를 호출하기 전에 빈B를 완전히 구성합니다. 다시 말해, 빈은 인스턴스화되고, 의존성이 설정되며, 관련 라이프사이클 메서드(구성된 초기화 메서드 또는 InitializingBean 콜백 메서드 등)가 호출됩니다.

### Examples of Dependency Injection

1. 생성자 기반 의존성 주입

```java
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // ...
}

<bean id="userService" class="com.example.UserService">
    <constructor-arg ref="userRepository"/>
</bean>

<bean id="userRepository" class="com.example.JdbcUserRepository"/>
```

`UserService` 클래스는 생성자를 이용해서 `UserRepository` 에 대한 의존성을 정읳고 있습니다. 생성자는 `UserRepository` 인스턴스를 매개변수로 받고, `userRepository` 필드로 할당합니다. 

XML 설정정보에서는, `userService` 빈을 정의하고, `<constructor-arg>` 속성을 통해 `userRepository` 빈을 주입해줍니다. 

2. Setter 기반 의존성 주입

```java
public class UserService {
    private UserRepository userRepository;
    
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    // ...
}

<bean id="userService" class="com.example.UserService">
    <property name="userRepository" ref="userRepository"/>
</bean>

<bean id="userRepository" class="com.example.JdbcUserRepository"/>
```

위의 예시에서는, `UserService` 클래스는 `UserRepository` 인스턴스를 받는  `setUserRepository()` setter 메서드가 있습니다. 스프링 설정정보 xml 에서는, `userService` 빈을 정의하고,  `userRepository` 빈을 `<property>` 속성을 통해서 주입하고 있습니다. 

3. 팩토리 메서드를 사용한 의존성 주입 

```java
public class UserServiceFactory {
    private UserRepository userRepository;
    
    public UserServiceFactory(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public UserService createUserService() {
        return new UserService(userRepository);
    }
}

<bean id="userServiceFactory" class="com.example.UserServiceFactory">
    <constructor-arg ref="userRepository"/>
</bean>

<bean id="userService" factory-bean="userServiceFactory" factory-method="createUserService"/>

<bean id="userRepository" class="com.example.JdbcUserRepository"/>
```

이 예시에서는, `UserServiceFactory` 클래스는 `UserService` 인스턴스를 만들기 위한 팩토리로 작동합니다. 팩토리 자체는 `UserRepository` 에 대한 의존성이 있고, 생성자를 통해 주입 받습니다. `createUserService()` 메서드는 `UserService` 인스턴스를 생성하고 반환합니다. 

스프링 설정정보 xml 에서는 `userServiceFactory` 빈을 정의하고, 생성자를 통해 `userRepository` 빈을 주입합니다. 그러면 `userService` 빈은 `factory-bean` 을 "userServiceFactory" , factory-method 를 "createUserService" 로 지정했기 때문에 생성됩니다.

## Autowiring Collaborators

스프링 컨테이너는 자동으로 협력하는 빈들 간의 관계를 연결(autowire)할 수 있습니다. `ApplicationContext`의 내용을 검사하여 Spring이 자동으로 빈의 협력자(다른 빈)를 해결하도록 할 수 있습니다. 자동 연결(Autowiring)에는 다음과 같은 장점이 있습니다:
- 자동 연결은 속성이나 생성자 인수를 지정해야 할 필요성을 크게 줄일 수 있습니다.
- 자동 연결은 객체가 진화함에 따라 구성을 업데이트할 수 있습니다. 예를 들어, 클래스에 의존성을 추가해야 하는 경우 구성을 수정할 필요 없이 해당 의존성이 자동으로 충족될 수 있습니다. 따라서 자동 연결은 개발 중에 특히 유용할 수 있으며, 코드 베이스가 안정화되면 명시적 연결로 전환하는 옵션을 무효화하지 않습니다.

XML 기반 구성 메타데이터를 사용할 때 `<bean/>` 요소의 `autowire` 속성을 사용하여 빈 정의의 자동 연결 모드를 지정할 수 있습니다. 자동 연결 기능에는 네 가지 모드가 있습니다. 빈마다 자동 연결을 지정하고 자동 연결할 빈을 선택할 수 있습니다. 다음 표는 네 가지 자동 연결 모드를 설명합니다:

| Mode          | Explanation                                                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `no`          | (기본값) 자동연결을 하지 않습니다. 빈 참조는 `ref` 속성을 이용해서 정의되어야 합니다. 기본 설정을 변경하는 것은 큰 프로젝트에서는 권장되지 않습니다. 왜냐하면 이 자체로 시스템의 구조를 문서화해주기 때문입니다.                  |
| `byName`      | 프로퍼티 이름으로 자동연결합니다. 스프링은 자동연결을 할 프로퍼티의 이름과 같은 빈을 찾습니다. 예를 들어, 빈의 정의가 이름을 통한 자동연결로 설정되어 있고 `master` 프로퍼티를 포함하면, 스프링은 `master` 이름을 가진 빈을 찾습니다. |
| `byType`      | 명시한 타입을 가진 정확히 한개의 빈이 컨테이너 내에 존재하면 자동연결합니다. 한 개 이상의 빈들이 존재하면, 치명적인 예외가 던져집니다. 매칭되는 빈이 없으면 아무 일도 발생하지 않습니다.                                  |
| `constructor` | `byType` 과 비슷하지만 생성자 인자들에 적용됩니다. 컨테이너에 생성자 인자 타입의 빈이 정확히 하나가 아닌 경우 치명적인 오류가 발생합니다.                                                          |

`byType` 또는 `constructor` 자동 연결 모드에서는 배열과 타입이 지정된 컬렉션(List, Set) 을 연결할 수 있습니다. 이러한 경우 의존성을 충족시키기 위해 예상 타입과 일치하는 컨테이너 내의 모든 자동 연결 후보가 제공됩니다.

만약 배열과 콜렉션 타입과 같은 의존성을 가지고 있다면, 스프링은 컨테이너 내에서 그 타입과 맞는 모든 빈들을 찾습니다. 그리고 매칭되는 모든 빈들이 주입됩니다. 예를 들어, 의존성 타입이  `List<MyService>` 라면, 스프링은 컨테이너 내에서 `MyService` 타입의 빈들을 모두 찾고, 리스트 내에 주입합니다. 

키 타입이 `String` 인 경우, 강력하게 타입이 지정된 `Map` 인스턴스를 자동 연결할 수 있습니다. 자동 연결된 `Map` 인스턴스의 값은 예상 타입과 일치하는 모든 빈 인스턴스로 구성되며, `Map` 인스턴스의 키에는 해당 빈 이름이 포함됩니다.

예를 들어, `Map<String, MyService>` 타입의 의존성이 있다면, 스프링은 `MyService` 타입의 빈들을 모두 찾고, `Map` 으로 주입합니다. `Map` 의 키 값은 빈의 이름이 되고, 밸류값은 `MyService` 빈 인스턴스가 됩니다. 

아래는 arrays, collections, maps 를 autowiring 한 예시입니다. 

```java
public class MyComponent {
	// array dependencies
    private MyService[] services;
	// list dependencies
    private List<MyService> serviceList;
    // map dependencies
    private Map<String, MyService> serviceMap;

    // Setters for autowiring
    public void setServices(MyService[] services) {
        this.services = services;
    }

    public void setServiceList(List<MyService> serviceList) {
        this.serviceList = serviceList;
    }

    public void setServiceMap(Map<String, MyService> serviceMap) {
        this.serviceMap = serviceMap;
    }
}
```

`byType` or `constructor`  autowiring 모드일때, 스프링은 컨테이너 내에  `MyService` 타입인 모든 빈들을 찾고, 해당 array, list, map 에 주입합니다. 

### Limitations and Disadvantages of Autowiring

Autowiring 은 프로젝트 내에서 일관되게 사용될 때 가장 효과적입니다. 일반적으로 자동 연결을 사용하지 않는 경우, 개발자가 한두 개의 빈 정의만 연결하기 위해 Autowiring을 사용하는 것은 혼란스러울 수 있습니다.

아래는 Autowiring의 단점입니다 :
- `property`와 `constructor-arg` 설정의 명시적 의존성은 항상 자동 연결보다 우선합니다. 기본 타입, `String`, `Class`(및 이러한 단순 속성의 배열) 같은 단순 속성은 자동 연결할 수 없습니다. 이 제한은 의도적으로 설계된 것입니다.
- Autowiring은 명시적 연결보다 정확성이 떨어집니다. 앞서 표에서 언급했듯이 Spring은 예기치 않은 결과를 초래할 수 있는 모호한 경우에 추측하지 않도록 주의하지만, Spring으로 관리되는 객체 간의 관계가 더 이상 명시적으로 문서화되지 않습니다.
- Spring 컨테이너에서 문서를 생성하는 도구에서 연결 정보를 사용하지 못할 수 있습니다.
- 컨테이너 내의 여러 빈 정의가 자동 연결될 setter 메서드나 생성자 인수에 의해 지정된 타입과 일치할 수 있습니다. 배열, 컬렉션 또는 `Map` 인스턴스의 경우 이는 반드시 문제가 되지는 않습니다. 그러나 단일 값을 예상하는 의존성의 경우 이러한 모호성은 임의로 해결되지 않습니다. 고유한 빈 정의를 사용할 수 없는 경우 예외가 발생합니다.

후자의 시나리오에서는 다음과 같은 몇 가지 옵션이 있습니다:

- 명시적 연결을 선호하여 자동 연결을 포기합니다.
- 다음 섹션에 설명된 대로 빈 정의의 `autowire-candidate` 속성을 `false`로 설정하여 자동 연결을 피합니다.
- `<bean/>` 요소의 `primary` 속성을 `true`로 설정하여 단일 빈 정의를 기본 후보로 지정합니다.
- 어노테이션 기반 컨테이너 구성에 설명된 대로 어노테이션 기반 구성에서 사용 가능한 더 세분화된 제어를 구현합니다.

### Excluding a Bean from Autowiring

빈 관점에서, 빈을 Autowiring 에서 제외할 수 있습니다. 스프링의 XML 포맷에서,  `<bean/>` 속성의  `autowire-candidate` 속성을 `false` 로 지정하면 됩니다. 컨테이너는 그 특정 빈의 정의를 autowiring infrastructure 가 사용하지 못하게 합니다. 

## Method Injection

대부분 어플리케이션 시나리오에서, 컨테이너 내 대부분의 빈들은 **싱글톤**입니다. 하나의 싱글톤 빈이 다른 싱글톤 빈과 협력하거나 non-싱글톤 빈이 다른 non-싱글톤 빈과 협력해야 할 때, 하나의 빈을 다른 빈의 property 로 정의함으로써 핸들링할 수 있습니다. 하지만 문제는 **빈의 라이프사이클이 다를 경우** 발생합니다. 예를 들어 싱글톤 빈A 가 non-싱글톤 빈B(prototype) 을 사용해야 할 때를 봅시다. 컨테이너는 싱글톤 빈A 를 오직 한번만 생성하고, properties 를 설정할 수 있는 기회는 한번밖에 없습니다. 컨테이너는 빈A에게 필요할 때마다 빈B의 인스턴스를 공급할 수 없습니다. 

해결책은 제어의 역전의 일부를 포기하는 것입니다. `ApplicationContextAware` 인터페이스를 구현하고, 컨테이너에 `getBean("B")` 호출을 하여 빈 A가 컨테이너를 인식하게 만들면, 빈 A가 필요할 때마다 (일반적으로 새로운) 빈 B 인스턴스를 요청할 수 있습니다. 다음 예제는 이 접근 방식을 보여줍니다:

```java
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

/**
 * A class that uses a stateful Command-style class to perform
 * some processing.
 */
public class CommandManager implements ApplicationContextAware {

	private ApplicationContext applicationContext;

	public Object process(Map commandState) {
		// grab a new instance of the appropriate Command
		Command command = createCommand();
		// set the state on the (hopefully brand new) Command instance
		command.setState(commandState);
		return command.execute();
	}

	protected Command createCommand() {
		// notice the Spring API dependency!
		return this.applicationContext.getBean("command", Command.class);
	}

	public void setApplicationContext(
			ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}
}
```

하지만 이 코드는 권장되지 않습니다. 왜냐하면 비즈니스 코드가 스프링 프레임워크와 결합되기 때문입니다. 따라서 **Method Injection** 은 이러한 케이스를 깔끔하게 처리할 수 있게 해줍니다. 

### Lookup Method Injection

이 방법은 컨테이너가 컨테이너에서 관리되는 빈들의 메서드를 다른 빈의 검색 결과를 반환하여  오버라이드할 수 있게 하는 방법입니다. 이 검색은 prototype 빈을 포함합니다. 스프링 프레임워크는 메서드를 오버라이드할 서브클래스를 **CGLIB 라이브러리**의 바이트코드 생성을 통해 구현합니다. 

위의 예시를 다시 구현해봅시다.

```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

	public Object process(Object commandState) {
		// grab a new instance of the appropriate Command interface
		Command command = createCommand();
		// set the state on the (hopefully brand new) Command instance
		command.setState(commandState);
		return command.execute();
	}

	// okay... but where is the implementation of this method?
	protected abstract Command createCommand();
}
```

더 이상 스프링 관련 import 를 안해도 됩니다 ! 따라서 CommandManager 는 스프링 의존관계 관련된 코드가 없습니다. 스프링 컨테이너는 `createCommad()` 메서드를 동적으로 오버라이드합니다. 

주입될 메서드를 포함하는 클라이언트 클래스(위 예시에서는 `CommandManager`)에서는, 주입되는 메서드는 아래의 형식을 따라야 합니다. 

```xml
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

메서드가 abstract 이면, 동적으로 생성된 서브클래스가 이 메서드를 구현합니다. 아니면, 동적으로 생성된 서브클래스가 오리지널 클래스에 정의된 구체적인 메서드를 오버라이드 합니다. 

```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
	<!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
	<lookup-method name="createCommand" bean="myCommand"/>
</bean>
```

commandManager 로 정의된 빈은 자신의 myCommand 빈의 새로운 인스턴스가 필요할 때마다 createCommand 메서드를 호출합니다. 빈을 싱글턴/프로토타입으로 정의할지는 신중해야 합니다. 


> **Singleton vs. Prototype?**
> 
> 빈이 싱글턴으로 정의되었으면, 스프링 IoC 컨테이너는 그 빈의 정의에 따라 오직 하나의 인스턴스만 생성합니다. 그 하나의 인스턴스는 싱글턴 빈들의 캐시에 저장되고, 이후에 모든 요청들에 그 인스턴스가 반환됩니다. 
> 
> 빈이 프로토타입으로 정의되었다면, 빈에 대한 각 요청마다 새로운 인스턴스가 생성되고 반환됩니다. 자바의 `new` 와 비슷합니다. 스프링 컨테이너는 프로토타입 빈의 생명주기를 관리하지 않습니다. 

어노테이션 기반 모델에서는, **@Lookup**  어노테이션을 사용할 수 있습니다. 

```java
public abstract class CommandManager {

	public Object process(Object commandState) {
		Command command = createCommand();
		command.setState(commandState);
		return command.execute();
	}

	@Lookup("myCommand")
	protected abstract Command createCommand();
}
```

myCommand 라고 검색 대상을 명시하지 않아도, 타켓 빈이 리턴 타입에 맞게 검색됩니다.

```java
public abstract class CommandManager {

	public Object process(Object commandState) {
		Command command = createCommand();
		command.setState(commandState);
		return command.execute();
	}

	@Lookup
	protected abstract Command createCommand();
}
```




## Reference
- https://docs.spring.io/spring-framework/reference/core/beans/dependencies.html
- https://stackoverflow.com/questions/17599216/spring-bean-scopes
- https://stackoverflow.com/questions/16058365/what-is-difference-between-sin
