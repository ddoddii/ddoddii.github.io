+++
author = "Soeun"
title = "[Java] Java의 자료구조"
date = "2023-08-30"
description = "Java Collection : Array, List, Set, Map"
categories = [
    "CS"
]
tags = [
    "Java", "자료구조"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/384bd77d-965b-4d8d-aa98-015cedc6ad55"
+++

## 1. Data Structure
자료구조는 컴퓨터 프로그램에서 데이터를 처리하기 위해 만든 구조로 **Array, List, Map**이 대표적인 형태이다. 

### 배열(Array)
배열은 가장 전통적이고 기본이 되는 자료 구조 입니다. 데이터를 순차적으로 저장해 `0부터 시작`하는 `인덱스`를 통해 접근한다. 

- 일반적으로 배열은 선언할때 크기가 고정됨.
- 데이터를 순차적으로만 접근할 수 있어 위치를 모르는 경우 효율이 떨어짐.
- 배열에 들어가는 데이터는 모두 동일한 자료형 이어야 함.
- 배열 중간에 값을 추가하려면 기존 데이터를 모두 이동시켜야 함.

자바 역시 배열은 가장 기본적인 자료형으로 프로그램 개발에 빼놓을수 없는 필수 요소 입니다. 그러나 구조에 따른 제약과 사용의 불편함등으로 인해 List를 더 많이 사용합니다. 그러나 순차적으로 사용하는 단순한 숫자나 문자등으로 이루어진 집합형 데이터의 처리에 있어 배열만큼 간단하고 빠른 자료구조는 없습니다.

### 리스트(List)
배열과 유사한 **순차적인 자료구조**를 제공 합니다. 데이터 접근을 위해 **인덱스를 사용**해야 하는 점은 배열과 같지만 배열의 모든 **문제점을 해결**하고 있습니다.

- 데이터 크기가 고정되지 않음.
- 데이터를 다루기 위한 여러 방법이 제공됨.
- 리스트의 데이터는 서로 다른 타입일 수 있음. -> 일관된 처리가 어려워 `보통은 동일하게` 처리함.
- 배열 중간에 값을 추가하거나 삭제하기 쉬움.
- 특정 데이터가 포함되어 있는지 확인은 가능하나 검색을 위해서는 별도 구현이 필요.

**Linked List**는 현재 데이터에 **다음** 데이터를 읽을 수 있는 정보를 추가한 것으로 불연속적으로 존재하는 데이터를 서로 연결할 수 있는 방법을 제공 합니다. **Double Linked List**는 이전과 다음 데이터 정보를 모두 가지고 있는 형태이며 자바의 경우 **LinkedList** 클래스가 제공되는데 실제로는 **Double Circular Linked List**(순환구조가 추가된 Double Linked List) 형태를 구현해 둔 것입니다.

### 맵 (Map)
데이터를 **Key:Value(키:값)** 의 쌍으로 저장하는 방식입니다. 
맵을 사용했을때 얻을 수 있는 가장 큰 장점은 원하는 데이터를 손쉽게 찾을 수 있다는 점입니다.

- 데이터를 저장할 때 해당 데이터를 찾기 위한 **Key**를 부여.
- **Key**값을 알면 언제든 쉽게 데이터를 찾을 수 있음.
- Value 에 **객체형**이 들어갈 수 있어 복잡한 데이터 처리가 가능.

### 이터레이터(Iterator)
이터레이터는 서로다른 자료구조(Vector, ArrayList, LinkedList)의 데이터를 동일한 방법으로 다음 데이터에 접근하는 방법을 제공하는 인터페이스로 자바 컬렉션 프레임워크의 일부 입니다.

## 2. Collection Framework
컬렉션 프레임워크는 자바에서 데이터를 저장하는 클래스들을 **표준화한 설계 구조**를 말합니다. 이러한 구조를 바탕으로 자바의 기본 자료구조 클래스들이 구성되어 있으며 체계화되고 **일관된 구조**를 가지게 되었습니다.

Collection, Map, List, Set 인터페이스를 중심으로 다음과 같은 클래스 계층구조를 형성하고 있습니다.

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/fc892a83-c063-4970-8c66-466a6b00ef20)

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/902bcad8-33e9-43f1-890a-6ea1d84f5371)



|인터페이스|설명|특징|대표 구현 클래스|
|---------|----|----|----------------|
|List|순서가 있는 데이터의 집합|데이터의 중복 혀용|ArrayList, LinkedList|
|Set|순서를 유지하지 않는 데이터 집합|데이터 중복 허용X|HashSet, LinkedHashSet|
|Map|키와 밸류의 쌍으로 이루어진 데이터 집합|순서 유지X, 키 중복X, 밸류 중복O|HashMap, LinkedHashMap, Properties

### Collection Interface
List 와 Set 의 상위 인터페이스이다. 즉, List 와 Set 를 구현한 모든 클래스들은 Collection 인터페이스의 메서드를 사용할 수 있으므로 구현클래스와 상관없이 동일한 방법으로 데이터를 다룰 수 있습니다.

**컬렉션 객체의 생성** 
```java
ArrayList<String> list = new ArrayList<>();  //권장 안됨
List<String> list = new ArrayList<>();  // 권장됨
```

```java
Collection<String> c1 = new HashSet<>();
Collection<String> c2 = Arrays.asList("three", "four"); 
Collection<String> c3 = Collections.singleton("five");
```

**컬렉션에 데이터 추가, 삭제**

데이터 추가는 `add(), addAll()` 메서드 사용한다. 지정된 타입의 데이터를 하나씩 추가할때는 `add()`, 컬렉션 타입을 추가할때는 `addAll()` 을 이용해 개별 원소를 꺼내어 추가하는 형식입니다.

```java
c1.add("one");
c1.addAll(c2);
```

데이터의 삭제는 `remove(), removeAll(), retainAll(), clear()` 등의 메서드를 사용 합니다. retailAll()의 경우 인자의 데이터를 제외한 나머지 데이터를 모두 삭제하는 메서드이고 clear() 는 모든 데이터를 삭제 합니다.

```java
c1.remove("one");
c1.remove("c2");
c1.retainAll("four");
c1.clear();
```

**컬렉션 데이터 확인 및 변환**

특정 데이터가 컬렉션 안에 존재 하는지 혹은 비어 있는지를 확인할 수 있으며 크기를 구할 수 있습니다. `containsAll()` 는 인자로 주어진 collection 과 모두 일치해야 true 를 반환한다. 

```java
c1.isEmpty();
c1.contains("zero");
c1.containsAll(c2);
```

**컬렉션 데이터의 사용**

컬렉션에 들어있는 데이터를 사용하기 위해서는 **다소 복잡한 과정**을 거쳐야 합니다. 컬렉션 자체는 구체적인 구현이 아니므로 직접적으로 데이터에 접근하는 방법은 포함되어 있지 않습니다.

List 나 Set 인터페이스를 이용해 처리하거나 Collection 인터페이스 타입으로 처리하려면 다음의 방법중 하나를 사용할 수 있습니다.

**a) 배열로 변환해서 for문과 사용**

특정 위치의 데이터를 직접 선택할 필요없이 순차적으로 모든 데이터를 사용하는 경우 단순하게 for 문만 사용해도 됩니다.

```java
for(String s : c1) {
    System.out.println(s);			
}
```

그러나 특정 위치의 데이터를 직접 다루고자 한다면 앞에서 다룬 `toArray()` 메서드를 이용해 배열로 변환해 사용하는 방법을 사용해야 합니다. 배열 데이터는 인덱스로 접근할 수 있으므로 각각의 데이터를 차례로 접근하거나 특정 위치 데이터를 직접 접근해 사용할 수 있습니다.

```java
String[] converted2 = c1.toArray(new String[c1.size()]);
for(int i=0; i < converted2.length; i++) {
    System.out.println(converted2[i]);
}
```

**b) Iterator 사용**

Iterator는 다음 데이터에 접근하는 방법을 제공하는 인터페이스 입니다. Collection 인터페이스는 Iterable 인터페이스를 상속받고 있으며 iterator() 메서드로 Iterator 객체를 구할 수 있습니다.

여기서는 모두 데이터 출력을 예시로 살펴보고 있지만 획득한 데이터를 조작하거나 다른 메서드의 인자로 전달하는등 필요한 작업에 활용할 수 있습니다.

```java
Iterator iter = c1.iterator();
while(iter.hasNext()) {
    System.out.println(iter.next());
}
```

**c) forEach()를 사용**

`forEach`는 for-loop를 좀 더 효율적으로 사용하는 방법으로 Java8에서는 람다식으로 표현이 가능합니다.

```java
c1.forEach(s -> System.out.println(s));
c1.forEach(System.out::println);
```

## 3. List, Set
> List 와 Set 은 모두 Collection 인터페이스를 상속받고 있으며 List 의 구현 클래스들은 AbstractList 클래스를 상속 받는 구조 입니다.

### List
List 인터페이스는 배열과 유사한 자료구조로 **중복이 허용**되면서 **저장순서가 유지**되는 구조를 제공 합니다. 구현 클래스로는 Vector, ArrayList, LinkedList 가 있으며 가장 널리 사용되는것은 **ArrayList** 입니다.

|Method|설명|
|---|----|
|add((index),val), addAll((index),collection)|새로운 요소 추가, 위치를 지정하거나 컬렉션 데이터를 한번에 추가 가능|
|get(index)|지정된 위치(index)에 있는 객체 반환|
|indexOf(val)|객체의 위치(index) 반환|
|lastIndexOf(val)|List의 마지막 요소부터 역방향으로 위치 반환|
|listIterator()|List의 객체에 접근할 수 있는 ListIterator를 반환(양방향-> next(), previous())|
|remove(val or index)|지정된 위치에 있는 객체 또는 해당 값 중 첫번째 값를 삭제하고 삭제된 객체를 반환|
|contains(val)|해당 값이 배열에 있는지를 검색해서 True/False 반환|
|set(index, val)|지정된 위치에 객체를 저장|
|get(index)|해당 index 에 있는 값 반환|
|indexOf(val)|해당 밸류가 어디 있는지 반환|
|subList(fromIndex, toIndex)|지정된 범위에 있는 객체를 새로운 List로 반환|
|sort()|지정된 Comparator 로 List 요소 정렬(기본은 오름차순) - 반대는 reverse()|
|toArray()|ArrayList 를 Array로 변환|
|asList()|Array 를 ArrayList 로 변환|

### ArrayList

```java
List<String> l1 = new ArrayList<>();
List<Stirng> l2 = Arrays.asList("one","two");
List<String> l3 = List.of("three","four");

l1.add("five");
l1.addAll(l2);
l1.set(0,"zero");

```

### LinkedList

컬렉션<> [이름] = new 세부class 로 생성했던 것과는 달리, LinkedList 는 컬렉션 대신 LinkedList 를 사용해야 한다. 

```java
LinkedList<Integer> queue = new LinkedList<Integer>();
queue.offer(11); //queue에 삽입
queue.offer(22); 
queue.poll();
queue.peek();

ListIterator<Integer> it = queue.listIterator();
if (it.hasNext()) {
	System.out.println(it.next());
}
```


### Set 
Set 인터페이스는 List와 유사하지만 **중복이 허용되지 않고** 기본적으로는 **순서가 유지 되지 않습니다**. 구현 클래스로는 HashSet, LinkedHashSet, EnumSet, TreeSet, CopyOnWriteArraySet 등이 있으며 가장 널리 사용되는것은 **HashSet** 입니다. 순서가 필요한 경우 **LinkedHashSet** 클래스나 SortedSet 인터페이스를 구현한 **TreeSet**등을 사용할 수 있습니다.

|Method|설명|
|---|----|
|add(), addAll()|기존에 없는 새로운 요소를 추가, 컬렉션 데이터를 한번에 추가하는 것도 가능|
|clear()|모든 요소를 삭제|
|contains(), containsAll()|인자의 객체를 포함하고 있는지 확인, 컬렉션 전체를 비교할수도 있음|
|isEmpty|요소가 하나도 없는지 확인|
|iterator()|현재 Set의 객체에 접근할 수 있는 Iterator를 반환|
|remove(), removeAll()|특정 객체를 삭제하거나 컬렉션 전체를 삭제|
|size()|Set 에 저장된 요소의 크기를 반환|
|toArray()|Set -> Array 변환|

### HashSet, LinkedHashSet
**가장 대표적인 Set** 구현 클래스 입니다. 데이터가 중복되지 않는 자료구조가 필요한 경우 사용할 수 있으며 LinkedHashSet 의 경우 데이터가 입력된 순서로 저장됩니다.

```java
Set<String> s1 = new HashSet<>();
Set<String> s2 = Set.of("three","four");

s1.addAll(Arrays.asList("one","two"));
s1.addAll(s2);
s1.add("five");
s1.add("two"); // 기존에 있으므로 새로 추가 안됨
s1.remove("five");
boolean check = s1.contains("one");
```

LinkedHashSet 의 경우 생성 클래스만 다르고 사용하는 방법은 기본적으로 동일 합니다

```java
LinkedHashSet<String> lset = new LinkedHashSet<>();
lset.addAll(Arrays.asList("one","two","three","four"));
lset.add("five");
```

Iterator 가 필요한 경우 다음과 같이 Iterator 객체를 가지고와 사용할 수 있습니다.
```java
Iterator<String> iter = lset.iterator();
while(iter.hasNext()) {
    System.out.println(iter.next());
}
```

### TreeSet
**SortedSet** 인터페이스를 구현한 클래스 입니다. HashSet과 동일하게 중복된 데이터를 저장할 수 없으며 LinkedHashSet이 입력한 순서로 저장되는것과 달리 오름차순으로 데이터를 정렬 합니다.

```java
Set<Integer> tset = new TreeSet<>();
tset.addAll(Arrays.asList(50,10,60,20));
```

## 4. Map
> Map 은 List 계열과 달리 순차적으로 데이터를 관리하지 않고 Key 와 Value 의 쌍으로 데이터를 관리합니다.

### Map 개요
Map은 Collection 인터페이스를 상속받지 않으며 그 자체로 인터페이스로 여러 Map 구현클래스를 가지고 있습니다. 가장 대표적인 클래스는 **HashMap** 입니다.

HashTable 과 HashMap 이 헷갈렸는데, 그냥 HashMap 을 쓰는 것이 가장 좋다 ! (HashTable은 옛날것..)

- Map은 Collection 인터페이스를 상속하지 않음.
- Key 집합은 Set 으로 볼 수 있음. (중복X)
- Value 집합은 Collection 으로 볼 수 있음.

Map<key, Value> -> key, value 의 type 은 상관 없다 !! 

```java
Map<String,String> map = new HashMap<>();
map.put("109875","홍길동");
map.put("109894","김사랑");
System.out.println(map.get("109894")); // 김사랑
```

- 타입 파라미터로 Key,Value 의 타입을 지정
- 문자열 이외 클래스 타입도 가능
- 필요한 데이터를 찾기 위해서는 Key 를 인자로 사용

다음과 같은 주요 메서드를 사용해 데이터를 추가, 검색, 삭제등의 작업을 수행할 수 있으며 특히 전체 데이터를 출력하거나 하는 경우 key, value 의 값을 각각 Set 과 Collection 타입으로 변환하는 메서드가 유용하게 사용 됩니다.


|Method|설명|
|---|----|
|put(key, value), putAll(Map)|새로운 요소를 추가, 데이터를 한번에 추가하는 것도 가능|
|get()|특정 Key 에 해당하는 값을 가지고 옴|
|remove(key)|Map 요소 삭제|
|entrySet()|key 와 value 값 모두를 Entry 객체(Set 형태)로 반환|
|keySet|key 요소만 Set 객체로 반환|
|size()|크기를 반환|
|values()|value 요소만 Collection 타입으로 변환|
|containsKey(key)|map의 요소 중 Key 를 포함하는지 True/False 반환|
|containsValue(value)|map 의 요소 중 value 가 있는지 True/False 반환|

## Reference
- https://dinfree.com/lecture/language/112_java_6.html
  - (내가 다시 볼 용도로 위의 페이지와 거의 동일함)