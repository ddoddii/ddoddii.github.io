+++
author = "Soeun"
title = "스프링 IoC Container 는 무엇이며 어떤 역할을 하는가?"
date = "2024-03-20"
summary = "Spring Core 1편 - IoC Container, Bean"
categories = [
    "CS"
]
tags = [
    "Spring"
]
slug = "ioc-container"
series = ["Spring Core"]
series_order = 1
+++

{{< alert icon="edit">}}
본 포스팅은 **Spring Core 공식 문서**를 번역하고, 추가적으로 제 생각을 덧붙인 글입니다.
{{< /alert >}}


## 1. Intro to IoC Container & Beans

스프링은 IoC 원칙을 구현하고 있습니다. Dependency Injection(DI) 는 IoC의 특별한 형태로, 오브젝트들이 자신의 의존성을 생성자, 팩토리 메서드, 오브젝트 인스턴스의 필드로만 정의하는 것입니다. IoC 컨테이너는 빈을 만들 때 이러한 의존성들을 주입해줍니다. 이 프로세스는 빈이 직접 스스로를 초기화하거나 의존성을 설정하는 것과는 반대 방향이므로, 제어의 역전이라 부릅니다. 

 `org.springframework.beans` 와 `org.springframework.context` 패키지는 스프링 프레임워크의 IoC 컨테이너의 기본입니다. [`BeanFactory`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 인터페이스는 어느 타입의 오브젝트든 관리할 수 있는 더욱 심화된 configuration을 제공합니다.   [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/ApplicationContext.html) 는 `BeanFactory` 의 sub-interface 입니다. ApplicationContext 는 몇가지의 추가적인 기능들을 더해줍니다. 

`BeanFactory` 는 configuration framework 와 기본적인 기능들을 제공하고, `ApplicationContext` 가 엔터프라이즈에 특화된 기능들을 추가해줍니다. `ApplicationContext` 는 완벽하게 `BeanFactory` 의 부분집합입니다.

스프링에서, IoC 컨테이너에 의해 관리되는 오브젝트들을 `bean` 이라고 부릅니다. 빈은 IoC 컨테이너에 의해 초기화되고, 모이고, 관리되는 오브젝트들입니다. 빈들과 그 빈들의 의존성들은 컨테이너가 사용하는 메타데이터에 반영되어 있습니다.

## 2. Container Overview

`org.springframework.context.ApplicationContext` 인터페이스는 IoC 컨테이너를 대표하고, 빈들을 초기화하고, 설정하고, 모으는 역할을 합니다. 컨테이너는 어떠한 오브젝트를 초기화하고, 설정하고, 모을지 configuration metadata 를 읽음으로써 알 수 있습니다. Configuration metadata 는 XML 또는 자바 어노테이션 혹은 자바 코드 형태로 표현됩니다. 이것은 내 어플리케이션을 구성하는 오브젝트가 무엇인지 표현하고, 그들간의 관계성을 표현할 수 있게 해줍니다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {  
    @Nullable  
    String getId();  
  
    String getApplicationName();  
  
    String getDisplayName();  
  
    long getStartupDate();  
  
    @Nullable  
    ApplicationContext getParent();  
  
    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;  
}
```

실제 `ApplicationContext` 인터페이스의 구현체 중 일부는 스프링에 내장되어 있습니다. Stand-alone 어플리케이션 같은 경우에는   [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) 또는 [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html) 의 인스턴스를 만듭니다. XML 이 전통적으로 메타데이터를 설정하는 방법이었지만, 자바 어노테이션을 사용해 컨테이너에게 알려줄 수도 있습니다.

대부분의 시나리오에서, 개발자는 스프링 IoC 컨테이너를 시작하기 위해 따로 코드를 명시적으로 작성하지 않아도 됩니다. 예를 들어, 웹 어플리케이션 시나리오에서는 web.xml 파일에 있는 8줄 정도의 보일러플레이트 설정이면 충분합니다.

아래는 고차원에서 본 스프링이 동작하는 원리입니다. 어플리케이션 클래스들과 메타데이터가 합쳐지고, ApplicationContext 가 생성되고 초기화되면 실행 가능한 어플리케이션이 생깁니다.


<img width="500" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/549659a7-2e65-4d3f-a6ae-e65126160e00">

### Configuration Metadata

위에서 봤듯이, 메타데이터는 굉장히 중요합니다. 메타데이터는 스프링 IoC 컨테이너에게 어플리케이션 내 오브젝트들을 어떻게 초기화하고, 구성하고, 모을지 말해줍니다. 메타데이터는 전통적으로 XML 포맷입니다.

그러나 XML이 메타데이터를 설정하는 유일한 방법인 것은 아닙니다. 스프링 IoC 컨테이너는 메타데이터가 쓰여진 포맷과 완전히 독립적입니다. 따라서 요즘은 많은 개발자들은  [Java-based configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html) 을 선택합니다.

스프링 컨테이너의 메타데이터 형태에는, 아래와 같은 형태들도 있습니다.

- [Annotation-based configuration](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html): 어노테이션 기반 설정

- [Java-based configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html): 자바를 사용한 빈 설정 ( [`@Configuration`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/annotation/Import.html),  [`@DependsOn`](https://docs.spring.io/spring-framework/docs/6.1.5/javadoc-api/org/springframework/context/annotation/DependsOn.html) annotations)

스프링 환경설정은 컨테이너가 관리해야 할 1개 이상의 빈의 정의들로 구성되어 있습니다. XML 기반의 메타데이터에서는 <bean/> 요소들로 설정합니다. 자바에서는 @Configuration 내 @Bean 어노테이션을 사용합니다.

이러한 빈 정의들은 실제 어플리케이션을 구성한느 실제 오브젝트들과 대응합니다. 실제로, 개발자는 서비스 레이어 오브젝트, 영속성 레이어 오브젝트(레포지토리, DAO), 컨트롤러 오브젝트, 인프라 오브젝트(JPA EntityManagerFactory) .. 등을 정의합니다.

아래는 기본적인 XML-기반 메타데이터의 구조입니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="..." class="...">
		<!-- collaborators and configuration for this bean go here -->
	</bean>

	<bean id="..." class="...">
		<!-- collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions go here -->

</beans>
```

- id 속성은 스프링 컨테이너 내에서 빈 정의를 구분하는 유니크한 스트링입니다. 다른 빈들은 id를 참고해서 빈에 대한 의존성을 판별합니다. 
- class 속성은 빈의 타입을 정의하고, 전체 클래스 이름을 사용합니다. 

id 속성의 값을 사용해서 의존관계에 있는 오브젝트들을 설정할 수 있습니다. 

### Instantiating a  Container

 `ApplicationContext` 생성자에 공급된 로케이션 경로는 컨테이너가 외부 리소스 (로컬 파일 시스템, Java ClassPath..) 로부터 메타데이터를 로드할 수 있게 하는 스트링입니다.

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

아래는 서비스 레이어 오브젝트들의 설정 파일 예시입니다. (services.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- services -->

	<bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
		<property name="accountDao" ref="accountDao"/>
		<property name="itemDao" ref="itemDao"/>
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions for services go here -->

</beans>
```

아래는 DAO 와 관련된 설정 파일 예시입니다.(daos.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		https://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="accountDao"
		class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
		<!-- additional collaborators and configuration for this bean go here -->
	</bean>

	<!-- more bean definitions for data access objects go here -->

</beans>
```


여기서 서비스 레이어는 `PetStoreServiceImpl` 클래스와 2개의 DAO (`JpaAccountDao` , `JpaItemDao`) 로 구성되어 있는 것을 알 수 있습니다. **property nam**e 요소는 자바 빈 속성의 이름을 가리키고, **ref** 요소는 스프링 컨테이너 내에 다른 빈을 참조할 때 사용합니다 . **id** 와 **ref** 간 연결은 오브젝트 간의 의존성을 나타냅니다. 

위의 예시에서, `PetStoreServiceImpl` 클래스는 `accountDao` 와 `itemDao` 2개의 의존성을 가지고 있습니다. 이것은 ref 속성을 통해 알 수 있습니다. 그 중 한개의 `property name` 은 "accountDao"이고, 이것은 `PetStoreServiceImpl` 클래스가  `setAccountDao(AccountDao accountDao)` 라는 setter 메서드를 가지고 있음을 나타냅니다. id, ref, property name 속성들을 가지고 스프링은 개발자가 빈들을 정의하고, 의존성을 설정할 수 있게 해줍니다. 

#### Composing XML-based Configuration Metadata

이러한 빈 정의들은 여러개의 xml 파일들에 정의될 수 있습니다. 때때로, 각 xml 설정은 어플리케이션 아키텍쳐 내에 하나의 논리적인 레이어 또는 모듈을 나타냅니다. 

이러한 xml 들에서 빈 정의들을 로드하기 위해 **application context constructor** 를 사용할 수 있습니다. 이 생성자는 여러 개의 Resource 경로들을 사용합니다. 대체재로, <import /> 를 사용하여 하나의 파일에서 다른 파일로 빈 정의들을 로드할 수 있습니다. 

```xml
<beans>
	<import resource="services.xml"/>
	<import resource="resources/messageSource.xml"/>
	<import resource="/resources/themeSource.xml"/>

	<bean id="bean1" class="..."/>
	<bean id="bean2" class="..."/>
</beans>
```

위의 예시에서, 외부 빈 정의들은 3개의 파일 (`services.xml`, `messageSource.xml`,  `themeSource.xml`) 로부터 로드되었습니다. 모든 경로들은 import를 하는 파일 경로에 상대적입니다. 이말은 `services.xml` 는 import 하는 파일과 같은 디렉토리 내에 있어야 합니다. 


### Using the Container

`ApplicationContext` 는 인터페이스로, 다른 빈들과 그들의 의존관계를 관리할 수 있는 기능들을 제공하는 더 발전된 팩토리입니다. `T getBean(String name, Class<T> requiredType)` 를 사용해서, 내가 정의한 빈들의 인스턴스를 가져올 수 있습니다. 

`ApplicationContext` 를 사용해서 빈 정의들을 읽고, 접근할 수 있습니다. 

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

## 3. Bean Overview

스프링 IoC 컨테이너는 하나 이상의 빈들을 관리합니다. 이러한 빈들은 개발자가 컨테이너에게 제공하는 설정 메타데이터를 가지고 생성됩니다. 

컨테이너 안에서, 이러한 빈 정의들은 `BeanDefinition` 오브젝트로 나타낼 수 있습니다. 이 오브젝트는 아래의 메타데이터를 가지고 있습니다 :
- A package-qualified class name : 빈의 실제 구현 클래스
- Bean behavioral configuration elements(빈 동작 구성 요소) : 빈이 컨테이너 내에서 어떻게 동작해야 하는지 명시합니다. (스코프, 라이프사이클 콜백..)
- References to other beans(다른 빈에 대한 참조) : 빈이 작업을 수행하는 데 필요한 다른 빈들에 대한 참조입니다. 이러한 참조를 협력자(collaborators) 또는 의존성(dependencies)이라고도 합니다.
- Other configuration settings(기타 구성 설정) : 새로 생성된 객체에 설정할 기타 구성 설정입니다. 예를 들어, 풀의 크기 제한이나 연결 풀을 관리하는 빈에서 사용할 연결 수 등을 설정합니다.

빈을 생성하는 방법에 대한 정보를 포함하는 빈 정의 외에도, `ApplicationContext` 구현체들은 컨테이너 외부에서 (사용자에 의해) 생성된 기존 객체의 등록을 허용합니다. 이는 `getBeanFactory()` 메서드를 통해 ApplicationContext의 `BeanFactory`에 접근함으로써 이루어지며, 이 메서드는 `DefaultListableBeanFactory` 구현체를 반환합니다. `DefaultListableBeanFactory`는 `registerSingleton(..)`과 `registerBeanDefinition(..)` 메서드를 통해 이러한 등록을 지원합니다. 그러나 일반적인 애플리케이션은 오로지 일반적인 빈 정의 메타데이터를 통해 정의된 빈들로만 작업합니다.


### Naming Beans

각 빈은 한개이상의 식별자가 있습니다. 이러한 식별자들은 컨테이너 내에 유일해야 합니다. 빈은 대체로 한개의 식별자를 가지고 있습니다. 하지만, 만약 한개 이상이 필요하다면, 엑스트라 식별자들은 가명(aliases)으로 취급됩니다.

XML 기반 설정에선, id 속성 또는 name 속성, 혹은 둘 다 사용합니다. id 는 컨테이너에 의해 유일성 제약 조건이 있습니다. 

만약 name 또는 id 를 설정하지 않으면, 컨테이너는 유니크한 이름을 만듭니다. 하지만 만약 빈을 이름을 사용해서 참조하고 싶다면, 무조건 이름을 지어줘야 합니다. 

#### Aliasing a Bean outside the Bean Definition

빈 정의 자체에서는, `id` 속성으로 지정된 최대 하나의 이름과 `name` 속성에 지정된 다른 이름들을 조합하여 빈에 대해 여러 개의 이름을 제공할 수 있습니다. 이러한 이름들은 동일한 빈에 대한 동등한 별칭이 될 수 있으며, 애플리케이션의 각 컴포넌트가 해당 컴포넌트 자체에 특정한 빈 이름을 사용하여 공통 의존성을 참조할 수 있도록 하는 등의 상황에서 유용합니다.

그러나 빈이 실제로 정의된 곳에 모든 별칭을 지정하는 것이 항상 적절한 것은 아닙니다. 때로는 다른 곳에서 정의된 빈에 대한 별칭을 도입하는 것이 바람직할 수 있습니다. 이는 일반적으로 구성이 각 하위 시스템 간에 분할되고 각 하위 시스템이 자체 객체 정의 집합을 가지고 있는 대규모 시스템에서 일반적입니다. XML 기반 구성 메타데이터에서는 `<alias/>` 요소를 사용하여 이를 수행할 수 있습니다. 다음 예제는 이를 수행하는 방법을 보여줍니다:

```xml
<alias name="fromName" alias="toName"/>
```

이러한 경우에는 한 컨테이너에서"fromName" 이라고 불리는 하나의 빈이 다른 컨테이너에서 "toName" 으로 불릴 수도 있습니다. 

자바 기반 설정에서는 @Bean 어노테이션을 사용하여 이 기능을 사용할 수 있습니다. 

### Instantiating Beans

빈 정의는 하나 이상의 오브젝트를 만드는 레시피입니다. 컨테이너는 요청을 받았을 때 레시피를 보고, 이 설정 메타데이터를 사용해서 실제 오브젝트를 만듭니다. 

XML 기반의 설정 메타데이터를 사용하면, 오브젝트의 타입(또는 클래스) 를 <bean /> 요소 내의 class 속성에 명시합니다. 이 class 속성은 필수적입니다. 이 class 속성을 아래 2가지 방법 중 하나로 사용할 수 있습니다 :
- 일반적으로, 컨테이너 자체가 생성자를 리플렉션을 통해 호출하여 빈을 직접 생성하는 경우에 생성할 빈 클래스를 지정하기 위해 사용됩니다. 이는 `new` 연산자를 사용하는 Java 코드와 어느 정도 동등합니다.
- 덜 일반적인 경우로, 컨테이너가 빈을 생성하기 위해 클래스의 `static` 팩토리 메서드를 호출할 때, 객체를 생성하기 위해 호출되는 `static` 팩토리 메서드가 포함된 실제 클래스를 지정하기 위해 사용됩니다. `static` 팩토리 메서드 호출에서 반환되는 객체 유형은 동일한 클래스일 수도 있고 완전히 다른 클래스일 수도 있습니다.

#### Instantiation with a Constructor

생성자 접근 방식으로 빈을 생성할 때, 모든 일반 클래스는 Spring에서 사용 가능하고 호환됩니다. 즉, 개발 중인 클래스는 특정 인터페이스를 구현하거나 특정 방식으로 코딩될 필요가 없습니다. 간단히 빈 클래스를 지정하는 것으로 충분합니다. 그러나 해당 빈에 사용하는 IoC 유형에 따라 기본(빈) 생성자가 필요할 수 있습니다.

Spring IoC 컨테이너는 사실상 관리하고자 하는 거의 모든 클래스를 관리할 수 있습니다. 이는 진정한 JavaBean으로 제한되지 않습니다. 대부분의 Spring 사용자는 기본(no-argument) 생성자와 컨테이너의 속성을 모델로 한 적절한 setter 및 getter만 있는 실제 JavaBean을 선호합니다. 컨테이너에  non-bean-style 클래스를 포함시킬 수도 있습니다. 예를 들어, JavaBean 규격을 절대적으로 준수하지 않는 레거시 커넥션 풀을 사용해야 하는 경우, Spring은 이를 관리할 수 있습니다.

XML 기반 설정 메타데이터에서 아래와 같이 빈 클래스들을 지정할 수 있습니다. 

```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

#### Instantiation with a Static Factory Method

정적 팩토리 메서드를 사용해서 만드는 빈을 정의할 때, `class` 속성으로 정적 팩토리 메서드를 포함하는 클래스를 지정하고, `factory-method` 속성으로 팩토리 메서드의 이름을 지정하면 됩니다. 그러면 이 메서드를 호출하고, 생성자를 통해 만든 것처럼 실제 오브젝트를 반환받을 수 있습니다. 

아래 빈 정의는 팩토리 메서드를 호출함으로써 생성되는 빈을 뜻합니다. 이 정의는 반환될 오브젝트의 타입(클래스) 를 지정하지 않고, 팩토리 메서드를 포함하는 클래스를 지정합니다. 아래 예시에서 `createInstance()` 메서드는 무조건 `static` 메서드여야 합니다. 

```xml
<bean id="clientService"
	class="examples.ClientService"
	factory-method="createInstance"/>
```

아래 코드는 위의 빈 정의를 위한 클래스의 예시입니다.

```java
// Responsible for creating and managing its own instance using a private constructor and static field
// singleton 
public class ClientService {
	private static ClientService clientService = new ClientService();
	private ClientService() {}

	// static factory method
	public static ClientService createInstance() {
		return clientService;
	}
}
```

스프링 컨테이너가 시작되고 빈들의 설정정보를 읽으면, `ClientService` 클래스를 로드합니다. 정적 필드들이 초기화될때, `ClientService` 클래스는 private static field 가 있으므로, private 생성자를 이용하여  `ClientService`의 인스턴스를 생성합니다. `ClientService` 클래스는 public static method 인  `createInstance()` 가 있어서, 이 메서드는 static `clientService` field 에 저장되어 있는  `ClientService` 인스턴스를 반환합니다. 

스프링 컨테이너가 "clientService" bean 을 인스턴스화 해야 할때, 스프링은 static factory method 인  `ClientService.createInstance()` 를 호출합니다. 스프링은 이 인스턴스를 "clientService" bean 에게 할당합니다. 스프링 컨테이너에게 "clientService" bean 요청이 들어오면, 스프링은 매번 동등한 `ClientService` 인스턴스를 반환해줍니다. 

#### Instantiation by Using an Instance Factory Method

정적 팩토리 메서드와 비슷하게, 인스턴스 팩토리 메서드를 사용한 인스턴스화는 새로운 빈을 생성하기 위해 컨테이너에서 새로운 빈을 생성하기 위해 기존 빈의 non-static 메서드를 호출합니다. 이 메커니즘을 사용하려면, `class` 속성은 비워두고 `factory-bean` 속성에 객체를 생성하기 위해 호출될 인스턴스 메서드를 포함하는 현재(또는 부모나 조상) 컨테이너의 빈 이름을 지정하세요. `factory-method` 속성을 사용하여 팩토리 메서드 자체의 이름을 설정하면 됩니다.

```xml
<!-- the factory bean, which contains a method called createClientServiceInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
	<!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
	factory-bean="serviceLocator"
	factory-method="createClientServiceInstance"/>
```

아래는 대응하는 클래스의 예시입니다.

```java
public class DefaultServiceLocator {

	private static ClientService clientService = new ClientServiceImpl();

	// non-static method
	public ClientService createClientServiceInstance() {
		return clientService;
	}
}
```

처음 스프링이 빈 설정정보를 로드할 때, 스프링은 첫번째로 기본 생성자를 사용하여 "serviceLocator" 팩토리 빈의 인스턴스를 생성합니다. "clientService" 빈이 요청되었을 때, 스프링은 "serviceLocator" 빈 인스턴스에 있는   `createClientServiceInstance()` 메서드를 사용하여 `ClientService` 인스턴스를 생성합니다. 

#### Determining a Bean’s Runtime Type

Runtime 시 빈의 타입을 결정하는 것은 사소하지 않은 문제입니다. 빈의 메타데이터에 정의된 클래스는 참조할 최초의 클래스일 뿐입니다. 정의된 팩토리 메서드 또한 FactoryBean 클래스는 런타임 시 빈이 다른 클래스 타입으로 바뀔 수 있게 합니다. 또한 인스턴스 레벨 팩토리 메서드에서는 아예 빈의 클래스 타입을 명시하지 않을 수도 있습니다. 또한, AOP 프록시를 통해 빈의 인스턴스를 인터페이스 기반의 프록시로 포장할 수도 있습니다. 

따라서 실제 런타임 시 빈의 타입을 찾는 것은 `BeanFactory.getType` 를 추천합니다. 이 메서드는 위의 모든 경우들을 고려해서 실제 런타임 시 빈의 타입을 알려줍니다. 





## Reference
- https://docs.spring.io/spring-framework/reference/core/beans.html






