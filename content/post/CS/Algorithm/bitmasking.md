+++
author = "Soeun"
title = "비트마스크(BitMask) 알아보기"
date = "2024-01-05"
summary = "비트마스킹 논리 연산과 실제 알고리즘 문제 풀이"
categories = [
    "CS"
]
tags = [
    "알고리즘"
]
image = ""
+++
{{< katex >}}


## 비트 논리 연산자 

| 연산자 | 논리       | 설명                                |
| ------ | ---------- | ----------------------------------- |
| &      | AND        | 두 비트 모두 1이면 1                |
| \|     | OR         | 두 비트 중 1개만 1이면 1            |
| ^      | XOR        | 두 비트가 서로 다르면 1             |
| ~      | NOT        | 비트 반전                           |
| <<     | LEFT SHIFT | 지정한 수 만큼 비트들 왼쪽으로 이동 |
| >>       | RIGHT SHIFT           | 지정한 수 만큼 비트들 오른쪽으로 이동                                    |

| A | B | ~A | A&B | A\|B | A^B |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 0 | 0 | 1 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 | 1 | 0 |

#### Shift Left, Shift Rignt

- `A << B`
	- A 를 B 비트만큼 왼쪽으로 밀기
	- `A x (2^B)`
	- 3<<4  : \\(11_2\\) 을  왼쪽으로 4만큼 미니까 \\(110000_2\\) = 16 + 32 = 48 이 된다. 또는 \\(3*2^{4}\\) = 48 이다.
- `A >> B`
	- A를 B비트만큼 오른쪽으로 밀기
	- `A / (2^B)`
	- 2 >> 4  = 2 / 2^4 = 2/16 = 1/8

## 비트마스크

> Bitmasking is an operation by which we allow only certain bits to pass through while blocking / masking the rest.

비트마스크는 비트 연산을 이용해 부분집합을 표현하는 것이다. 
- 비트마스트를 통해 집합을 정수로 나타낼 수 있다. 
- 단, 이때 집합에 저장할 수 있는 수의 범위가 정해져 있어야 한다.
- 따라서 보통 **0부터 N-1까지 N개의 정수로 이루어진 집합**을 나태낼 때 사용한다.

ex ) N = 5 일 때, s = {1,3,4} 라는 집합에서 각 수 의 존재여부는 아래와 같이 나타낼 수 있다. 

| 4   | 3   | 2   | 1   | 0   |
| --- | --- | --- | --- | --- |
| 1    | 1    | 0    | 1    | 0    |

정수로 나타내면, \\(11010_2\\) = 26 이다. 배열을 나타낼 때 훨씬 공간을 적게 사용할 수 있다. 또한 연산의 시간복잡도가 O(1) 로 매우 빠르다. 

### 비트마스크의 연산 

집합 S 에 대해 수 X 를 검사, 추가, 삭제, 토글하는 방법

1. S 에 X 가 있는지 검사
   - S의 X번째 비트가 1 인지 0 인지 검사
   - X번째 비트만 1로 두고(비트마스킹) AND 연산 수행
   - `S & (1<<X)`

2. S 에 X를 추가 
   - S의 X번째 비트를 1로 변경 
   - X번째 비트만 두고 OR 연산 수행
   - `S | (1 << X)`

3. S에서 X를 삭제
   - S의 X번째 비트를 0으로 변경
   - X번째 비트만 0으로 두고 AND 연산 수행
   - `S & ~(1 << X)`

4. S에서 X를 토글
   - S에서 X번째 비트가 0이면 1로, 1이면 0으로 변경
   - X번째 비트만 1로 두고 XOR 연산 수행 
   - `S ^ (1 << X)`


## 실제 문제 적용
[프로그래머스 불량 사용자](https://school.programmers.co.kr/learn/courses/30/lessons/64064)

```python
import re
def solution(user_id, banned_id):  
    answer = set()  
    # wildcard 인 . 으로 교체
    banPatterns = [x.replace("*", ".") for x in banned_id]  
    search(0, 0, user_id, answer, banPatterns)   
    return len(answer)  
  
  
def search(idx, visit, userId, answer, banPatterns):  
    if idx == len(banPatterns):  
        answer.add(visit)  
        return  
  
    for i in range(len(userId)):  
        if (visit & (1 << i)) > 0 or not re.fullmatch(banPatterns[idx], userId[i]):  
            continue        
        search(idx + 1, visit | (1 << i), userId, answer, banPatterns)
```

예시) user_id = ["frodo", "fradi", "crodo", "abc123", "frodoc"], banned_id = ["*rodo", "*rodo", "******"]  

정답) 2 -> banned_id 에 유저가 대응되는 경우는 2가지이다. 

#### 풀이
- re 모듈을 사용하기 위해, * 를 regex 의 wildcard 인 . 으로 교체했다. 
- `search` 에서 DFS 를 사용하였다. 
- idx 는 banPattens 의 인덱스, i 는 user_id 의 인덱스를 뜻한다. visit bitmasking 을 저장하기 위한 값이다. 

**Bitwise AND** : `visit & (1 << i)` 
- 특정 유저 id (at index 'i') 가 이전에 체킹되었는지 확인한다. 
- visit 의 \\(i^{th}\\) bit 이 0 (아직 체킹 안됨) 이라면, `visit & (1 << i)`  의 결과는 0 이 된다. (x & 0 = 0) 
- 만약 체킹 안된 경우 (`visit & (1 << i)` 의 결과가 0 초과) 이거나(or) (banPattern[idx], user_id[i]) 가 일치하지 않으면 continue 한다. 

**Bitwise OR** :  `visit | (1 << i)` 
- 특정 유저 id (at index 'i') 가 사용되었음을 마킹하기 위함이다. 
- 1<< i 는 1을 i 비트만큼 왼쪽으로 민다. 
- OR 을 사용하면, visit 의  \\(i^{th}\\) bit 은 1이 된다. (x | 1 = 1) 
- ex) visit = 5 (\\(101_2\\) ) 이고 i = 1 이면, visit | (1<<1) = 101 | 010 = 111

1. Function Initialization 
   
   처음에 `search()` 는 idx = 0, visit = 0 으로 시작한다. 
   
2. Starting the search
   
   visit 는 bitmask 를 나타내기 위한 정수이다. 
   
3. Recursive Search 
   
   함수는 각 user_id 를 돌면서 현재 `bitPatterns[idx]` 와 매칭하는지 여부를 검사한다. 예를 들어, 첫번째 패턴 ".rodo" 는 "frodo" 와 "crodo" 와 매칭한다. 첫번째로 "frodo" 와 매칭했다고 하자. 
   
4. Updating Bitmask
   
   - "frodo" 와 매칭되면, `if (visit & (1 << i)) > 0 or not re.fullmatch(banPatterns[idx], userId[i])` 가 False 가 되므로 continue 되지 않고 아래 `search()` 함수로 넘어간다. 
   - "frodo" 가 첫번째 user_id 이므로, visit = 1 이 된다. 

5. Continuing the Search 
   
   - 'idx' 도 1 이 되었으므로, bitPatterns 의 다음 패턴에 대해서도 서치가 진행된다. 
   - 다음 패턴인 ".rodo" 는 "crodo" 와 매칭할 수 있다. ("frodo"는 이미 사용됨 - visit = 1, i = 0 이므로 `visit & (1 << i)`  에서 1 & 1 = 1 로 참이 되어 continue 가 실행된다)
   - "crodo"(i=2) 와 매칭한 후에는, visit = `visit | (1 << i)`  = 1 | (1 << 2) = 001 | 100 = 101 가 된다. 

6. Completing one combination 
   - 마지막 패턴 "......" 은 6자리 user_id 아무거나 매칭할 수 있다. 예를 들어 "abc123" 과 매칭되었다고 하자. 
   - 매칭 이후, visit = (5 | 1<<3) = 13 = \\(1101_2\\(이 된다. 즉 현재 패턴으로, user_id 에 있는 1,3,4 번째가 매칭되었다는 의미이다. 
   - 이 idx = 3 이 되었으므로, 13 은 answer 에 추가된다. 

7. Exploring other combinations
   - 함수는 다른 조합들도 모두 탐색하고 idx = 3 이면 visit 을 answer 에 추가한다. 이때 answer 가 set() 이므로, 중복되지 않는다. 
   - anwer 의 길이가 중복되지 않는 조합의 수이다. 


## Reference
- https://www.youtube.com/watch?v=xFWgZ5DTjFo
- https://gyyeom.tistory.com/62


