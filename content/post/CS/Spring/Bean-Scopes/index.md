+++
author = "Soeun"
title = "스프링빈의 스코프는 무엇인가?"
date = "2024-03-28"
summary = "Spring Core 3편 - Bean Scope"
categories = [
    "CS"
]
tags = [
    "Spring"
]
slug = "bean-scopes"
series = ["Spring Core"]
series_order = 3
draft=true
+++

{{< alert icon="edit">}}
본 포스팅은 **Spring Core 공식 문서**를 번역하고, 추가적으로 제 생각을 덧붙인 글입니다.
{{< /alert >}}


빈의 정의를 만드는 것은 실제 인스턴스를 만들기 위한 레시피를 생성하는 것입니다. 빈이 레시피라는 것은, 그 빈을 가지고 실제로 여러 개의 오브젝트 인스턴스를 만들 수 있음을 뜻합니다. 

빈에 의해 만들어지는 오브젝트에 필요한 의존성들과 설정을 구상할 수 있을 뿐만 아니라, 그 빈에 의해 만들어지는 오브젝트의 범위(scope) 를 설정할 수 도 있습니다.  이 방식의 접근은 굉장히 유연한데, 왜냐하면 실제 자바 클래스 레벨에서 스코프를 설정하는 것이 아니라 인스턴스를 만들기 전에 설정을 통해 스코프를 설정할 수 있기 때문입니다. 스프링 프레임워크에서는 6가지 스코프를 지원합니다. 그 중 4가지(request, session, application, websocket)는 web-aware ApplicationContext 를 사용할 때만 사용가능합니다. 

> **Bean Scope** 란 ? 
>
> 스프링 빈이 어플리케이션이 실행되는 동안 의 존재할 수 있는 범위입니다. 

> **web-aware ApplicationContext** 란?
> 
> ApplicationContext 의 특별한 형태로, 웹 어플리케이션 환경에서 작동하도록 만든 형태입니다. ApplicationContext 와 더불어 추가적인 기능들을 제공합니다. 

| Scope       | Description                                                                                                                     |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------- |
| singleton   | (Default) 각 스프링 IoC 컨테이너에 빈 정의 하나 당 하나의 오브젝트 인스턴스를 생성합니다.                                                                       |
| prototype   | 빈 정의 하나당 무한 개의 오브젝트 인스턴스를 생성합니다. 즉, 요청이 들어올 때마다 새로운 인스턴스를 생성합니다.  (Java 의 `new` 와 비슷)                                           |
| request     | 하나의 HTTP request 의 생명주기와 같은 스코프로 빈을 생성합니다. 각 HTTP request 는 빈 오브젝트 인스턴스 한개를 가지고 있습니다. web-aware ApplicationContext 일때만 사용가능합니다. |
| session     | HTTP Session 의 생명주기와 같은 스코프로 빈을 생성합니다. web-aware ApplicationContext 일때만 사용가능합니다.                                                |
| application | ServletContext 와 같은 생명주기로 빈을 생성합니다. web-aware ApplicationContext 일때만 사용가능합니다.                                                   |
| websocket   | WebSocket 와 같은 생명주기로 빈을 생성합니다. web-aware ApplicationContext 일때만 사용가능합니다.                                                        |

## The Singleton Scope

하나의 인스턴스만 생성되고, 모든 요청은 그 인스턴스를 사용합니다. 하나의 빈을 정의하고, 싱글턴으로 설정하면, 스프링 IoC 컨테이너는 오직 그 빈 정의를 가지고 하나의 오브젝트 인스턴스만 생성합니다. 싱글턴 인스턴스는 싱글턴 빈들의 캐시에 저장되고, 이후 그 빈에 대한 모든 요청과 참조는 캐시된 오브젝트를 반환합니다. 

<img width="624" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e815157e-af05-413a-98b7-40798e5738f2">

스프링의 싱글턴 빈 컨셉은 GoF 디자인패턴 책에 소개된 싱글턴 패턴과는 다릅니다. 싱글턴 디자인 패턴에서는, 오로지 하나의 인스턴스만 생성되도록 하드코딩 합니다. 반면, 스프링의 싱글턴 스코프는 각 컨테이너 당, 각 빈 당에 해당합니다. 스프링 컨테이너 내에서 빈을 싱글턴으로 정의하면, 컨테이너는 그 빈 정의를 가지고 오직 하나의 인스턴스를 생성합니다. 싱글턴 스코프는 빈 스코프의 기본값 입니다. 

xml 로 정의하려면, 아래와 같이 정의할 수 있습니다. 

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```


## The Prototype Scope

싱글턴이 아닌, 프로토타입 스코프는 요청이 들어올 때마다 새로운 빈의 인스턴스를 생성합니다. 다른 빈에게 주입되는 빈 또는 `getBean()` 을 통해 빈을 요청하는 것이 이것에 해당합니다. Stateful 빈은 프로토타입으로 설정하고, Stateless 빈은 싱글턴으로 설정하는 것이 좋습니다. 

----

> **왜 Stateful 빈은 프로토타입으로 설정하고, Stateless 빈은 싱글턴으로 설정하는 것이 좋을까요?**
> 
> Stateful 빈은 상태를 유지하는 빈입니다. 각 인스턴스는 자신의 속성에 대해  다른 값을 가지고 있을 수 도 있습니다. 상태를 가지고 있다는 것은 각 클라이언트 혹혹은 컨텍스트마다 상태가 다를 수 도 있습니다. 프로토타입으로 설정하면, 각 요청마다 독립적인 인스터스의 빈을 가집니다. 따라서 다수의 빈 인스턴스들이 각자의 상태를 가지고 공존할 수 있게 해줍니다. 
> 
> Stateless 빈은 상태를 유지하지 않는 빈입니다. 이러한 빈들은 주어진 input 값에만 따라 로직을 수행합니다. 따라서 어플리케이션 내 다른 부분들과 공유될 수 있습니다. 싱글턴 빈은 인스턴스를 오로지 하나만 만드므로, 리소스를 더욱 효율적으로 사용할 수 있습니다. 

실질적인 예시를 하나 더 봅시다. 

```java
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<Item> items = new ArrayList<>();

    public void addItem(Item item) {
        items.add(item);
    }

    public void removeItem(Item item) {
        items.remove(item);
    }

    public List<Item> getItems() {
        return items;
    }
}
```

위는 프로토타입 빈으로, 요청마다 새로운 인스턴스를 만듭니다. 각 사용자의 쇼핑카트는 자신만의 아이템들을 가지고 있습니다. 

```java
@Component
public class PriceCalculator {
    public double calculateTotalPrice(List<Item> items) {
        return items.stream()
                .mapToDouble(Item::getPrice)
                .sum();
    }
}
```

반면 위의 PriceCalculator 는 싱글턴 빈입니다. 이 빈은 어떠한 상태도 가지지 않고, 주어진 input 값에 대해서만 로직을 수행합니다. 

---

<img width="628" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6034f0c1-35ae-43d8-b74e-5571ef48e104">

다른 스코프들과는 달리, 스프링은 프로토타입 빈의 생명주기를 관리하지 않습니다. 컨테이너는 초기화하고, 설정하고, 클라이언트에게 건내주지만, 그 후로는 관리하지 않습니다. 프로토타입 빈들에 대해서는 destruction lifecycle callbacks 는 호출되지 않습니다. 직접 코드 상에서 프로토타입 스코프의 오브젝트를 정리해야 하고, 할당된 리소스를 해제해야 합니다. 스프링 컨테이너가 프로토타입 빈의 리소스를 관리하게 하려면, 커스텀 bean post-processor 를 정의할 수 도 있습니다. 

## Singleton Beans with Prototype-bean Dependencies

 프로토타입에 의존하는 싱글턴 빈을 사용할 때는, 의존성이 초기화 때 주입된 다는 것을 알아야 합니다. 프로토타입 빈을 싱글턴 빈에게 주입할 때, 새로운 프로토타입 빈 인스턴스가 주입됩니다. 

만약 실행시에 싱글턴 빈이 프로토타입 빈의 새로운 인스턴스를 얻기를 원한다면 어떻게 해야 할까요? 싱글턴 빈에 대한 의존성 주입은 단 한번 이루어지기 때문에 또 다시 주입할 수는 없습니다. 이때 **Method Injection** 를 활용할 수 있습니다. 



## Reference
- https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html



