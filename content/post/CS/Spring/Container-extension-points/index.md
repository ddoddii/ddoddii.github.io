+++
author = "Soeun"
title = "스프링 컨테이너의 확장은 어떻게 하는가?"
date = "2024-03-29"
summary = "Spring Core 4편 - Container Extension Points"
categories = [
    "CS"
]
tags = [
    "Spring"
]
slug = "container-extension-points"
series = ["Spring Core"]
series_order = 4
+++

{{< alert icon="edit">}}
본 포스팅은 **Spring Core 공식 문서**를 번역하고, 추가적으로 제 생각을 덧붙인 글입니다.
{{< /alert >}}

스프링 컨테이너의 확장이란, 빈 호출 과정에서 전/후/프로퍼티 설정 등의 '과정 자체'를 컨테이너가 수정/추가하여 확장할 수 있다는 의미입니다. 컨테이너를 확장하는 방법들을 알아봅시다.

## Customizing Beans by Using a `BeanPostProcessor`

`BeanPostProcessor` 인터페이스는 커스텀 초기화 로직, 의존성 주입 로직을 구현할 수 있는 콜백 메서드들을 정의하고 있습니다. 스프링 컨테이너가 초기화 끝난 후 커스텀 로직을 구현하고 싶다면, `BeanPostProcessor` 구현을 추가하면 됩니다.

```java
package org.springframework.beans.factory.config;

public interface BeanPostProcessor {  
    @Nullable  
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
        return bean;  
    }  
  
    @Nullable  
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
        return bean;  
    }  
}
```

여러 개의 `BeanPostProcessor` 인스턴스를 설정할 수 있고, `order` 속성을 통해 실행 순서도 제어할 수 있습니다. 이 경우는 BeanPostProcessor 가 Ordered 인터페이스를 구현할 때만 설정할 수 있습니다. 

```java
public interface Ordered {  
    int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;  
    int LOWEST_PRECEDENCE = Integer.MAX_VALUE;  
  
    int getOrder();  
}
```

`org.springframework.beans.factory.config.BeanPostProcessor` 인터페이스는 2개의 콜백 메서드를 포함하고 있습니다. 클래스가 컨테이너 내에 post-processor 로 등록되어 있을 때, 컨테이너가 만드는 각 빈 인스턴스에 대해 post-processor 는 초기화 메서드 전과 빈 초기화 후,  컨테이너로부터 콜백을 받습니다. post-processor 는 빈에 대해 어떠한 액션(콜백을 무시하는 것도 포함)도 취할 수 있습니다. post-processor는 콜백 인터페이스를 확인한 후, 빈을 proxy 로 래핑합니다. 

ApplicationContext 는 설정 메타데이터에서 `BeanPostProcessor` 인터페이스를 구현하고 있는 빈들을 자동으로 감지합니다. ApplicationContext는 이러한 빈들을 다른 빈 생성 이후에 호출할 수 있도록 post-processors 로 등록합니다. 

### Example: Hello World, `BeanPostProcessor`-style

아래는 컨테이너가 빈을 만들 때마다 toString() 메서드를 호출하는 커스텀 `BeanPostProcessor`  를 나타냅니다. 

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

	// simply return the instantiated bean as-is
	public Object postProcessBeforeInitialization(Object bean, String beanName) {
		return bean; // we could potentially return any object reference here...
	}

	public Object postProcessAfterInitialization(Object bean, String beanName) {
		System.out.println("Bean '" + beanName + "' created : " + bean.toString());
		return bean;
	}
}
```


## Customizing Configuration Metadata with a `BeanFactoryPostProcessor`

`org.springframework.beans.factory.config.BeanFactoryPostProcessor` 는 앞에 봤던  `BeanPostProcessor` 와 유사하지만, 한 가지 큰 차이점이 있습니다. `BeanFactoryPostProcessor`는 빈 설정 메타데이터에 대해 동작합니다. 스프링 IoC 컨테이너는 `BeanFactoryPostProcessor` 가 설정 메타데이터를 읽고, 빈을 초기화 하기도 전에 바꿀 수 있도록 허용합니다. 

`BeanFactoryPostProcessor` 는 ApplicationContext 내에 선언되면 자동으로 실행됩니다. 스프링은 이미 `PropertyOverrideConfigurer` 과`PropertySourcesPlaceholderConfigurer` 과 같이 사전 정의된 bean factory post-processors 가 있습니다. 

### Example: The Class Name Substitution `PropertySourcesPlaceholderConfigurer`

`PropertySourcesPlaceholderConfigurer` 를 사용해서 빈 정의를 외부 Properties 포맷으로 정의된 프로퍼티 값에서 가져와 정의할 수 있습니다. 이렇게 하면 외부 환경 관련 정의들(DB URLs, 비밀번호 등)을 커스텀할 수 있습니다. 

아래는 XML 기반 메타데이터 예시입니다. 

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
	<property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${jdbc.driverClassName}"/>
	<property name="url" value="${jdbc.url}"/>
	<property name="username" value="${jdbc.username}"/>
	<property name="password" value="${jdbc.password}"/>
</bean>
```

아래 Properties 파일에 정의된 값을 실행시에 `PropertySourcesPlaceholderConfigurer` 가 가져와서 데이터소스에 집어 넣습니다. 

```yaml
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

${jdbc.username} 는 실행시에 실제 값으로 대체됩니다. 


## Customizing Instantiation Logic with a `FactoryBean`

`org.springframework.beans.factory.FactoryBean` 을 사용하여 팩토리인 오브젝트를 구현할 수 있습니다. 

`FactoryBean` 인터페이스는 Spring IoC 컨테이너의 인스턴스화 로직에 플러그인으로 사용될 수 있는 지점입니다. 복잡한 초기화 코드를 (잠재적으로) 장황한 XML 대신 Java로 표현하는 것이 더 나은 경우, 사용자 정의 `FactoryBean`을 만들고 해당 클래스 내부에 복잡한 초기화 코드를 작성한 다음, 사용자 정의 `FactoryBean`을 컨테이너에 플러그인할 수 있습니다.

`FactoryBean<T>` 인터페이스는 다음 세 가지 메서드를 제공합니다:
- `T getObject()`: 이 팩토리가 생성하는 객체의 인스턴스를 반환합니다. 이 팩토리가 싱글톤을 반환하는지 프로토타입을 반환하는지에 따라 인스턴스가 공유될 수 있습니다.
- `boolean isSingleton()`: 이 `FactoryBean`이 싱글톤을 반환하면 `true`를, 그렇지 않으면 `false`를 반환합니다. 이 메서드의 기본 구현은 `true`를 반환합니다.
- `Class<?> getObjectType()`: `getObject()` 메서드가 반환하는 객체 타입을 반환하거나, 타입을 미리 알 수 없는 경우 `null`을 반환합니다.

`FactoryBean`이 생성한 빈 대신 실제 `FactoryBean` 인스턴스 자체를 컨테이너에 요청해야 하는 경우, `ApplicationContext`의 `getBean()` 메서드를 호출할 때 빈의 `id` 앞에 앰퍼샌드 기호(`&`)를 접두사로 사용하면 됩니다. 따라서 `id`가 `myBean`인 주어진 `FactoryBean`의 경우, 컨테이너에서 `getBean("myBean")`을 호출하면 `FactoryBean`의 생성물이 반환되는 반면, `getBean("&myBean")`을 호출하면 `FactoryBean` 인스턴스 자체가 반환됩니다.



## Reference
- https://docs.spring.io/spring-framework/reference/core/beans/factory-extension.html






