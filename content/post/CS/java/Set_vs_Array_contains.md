+++
author = "Soeun"
title = "[Java] HashSet vs. Arrays contains() 의 시간복잡도 비교"
date = "2023-10-13"
description = "많이 쓰는 contains() 메서드"
categories = [
    "CS"
]
tags = [
    "Java", "자료구조"
]
image = "https://github.com/ddoddii/ddoddii.github.io/assets/95014836/384bd77d-965b-4d8d-aa98-015cedc6ad55"
+++

Character 를 원소로 가진 배열A 가 있다고 하자. 이 character 안에 vowel 이 몇개 있는지 보려면 어떻게 해야 할까? 

```java
String s = "soeun";
char[] strArr = s.toCharArray();
int count = 0;
//sol1. Using ArrayList
List<Character> vowels = new ArrayList<Character>(Arrays.asList('a','e','i','o','u'));

//sol2. Using HashSet
Set<Character> vowels = new HashSet<>(Arrays.asList('a','e','i','o','u'));

for (int i = 0 ; i < k ; i++){
    if (vowels.contains(strArr[i])){
        count++;
    }
}
```
vowels 를 ArrayList 또는 HashSet 에 저장한 다음, contains 메서드를 쓰면 될 것이다. ArrayList 와 HashSet 모두 contains 메서드를 지원한다. 그러면 두 개의 시간 복잡도는 어떻게 될까? 

## HashSet.contains()

HashSet 은 HashMap 을 바탕으로 구현되었다. 이 contains() 메서드는 `HashMap.containsKey(object)` 메서드를 call 한다. 여기서는 object 가 Map 안에 있는지를 체크하는 것이다. 내부 map 은 데이터를 bucket 이라고 알려진 node 에 저장한다. 각 bucket 은 `hashCode()` 메서드를 사용해서 만들어진 hash code 에 대응한다. 

다음으로는 시간 복잡도에 대해 보자. **HashSet 의 contains() 메서드는 O(1) 의 시간복잡도**를 가진다. object 의 bucket 위치를 찾는 것은 O(1) 의 연산이다. 만약 collision 이 발생한다면, 최대 log(n) 의 시간복잡도를 가질 수 있는데, 이것은 bucket 의 내부 구조가 TreeMap 구조이기 때문이다. Java 7 은 내부 bucket 구조로 LinkedList 구조를 썼지만, 그 이후에는 TreeMap 을 쓴다. 하지만 대체적으로 hash code collision 은 희귀하다. 그래서 시간복잡도는 O(1) 이다. 

## ArrayList.contains()

ArrayList 는 object 가 list 안에 있는지를 찾을 때 `indexOf(object)` 메서드를 이용한다. 이 메서드는 전체 array 를 iterate 하면서 각 요소와 찾으려는 object 를 `equals(object)` 메서드를 이용해서 비교한다. 

시간 복잡도를 보면, ArrayList 의 contains() 메서드는 O(n) 의 시간복잡도를 가진다. 왜냐하면 전체 list 를 iterate 하므로, 시간복잡도는 list 내에 있는 원소의 개수에 비례하여 증가한다. 

iterations = 10,000 했을 때 실제 성능 테스트를 한 것의 결과이다. HashSet 의 결과가 훨씬 좋은 것을 알 수 있다. 

<img width="580" alt="image" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e886611c-8ebb-4d6d-a5ee-b2bc2ebb3920">


## Reference
- https://www.baeldung.com/java-hashset-arraylist-contains-performance