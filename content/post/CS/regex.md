+++
author = "Soeun"
title = "정규표현식 한번에 정리하기"
date = "2023-12-20"
summary = "Regex 101"
categories = [
    "CS"
]
tags = [
    "정규표현식"
]
slug = "regex-101"
+++
{{< katex >}}

## Regular Expressoin HOWTO

python 에서는 `re` 모듈을 사용하여 정규표현식(regex pattern)을 사용할 수 있다. 
 
### Simple Patterns

정규표현식 'test' 는 오로지 test 만과 매칭된다. 하지만 자기 자신과 매칭되지 않는 문자들이 있는데, 이것을 metacharacters  라고 한다. 종류는 다음과 같다.
```
. ^ $ * + ? { } [ ] \ | ( )
```

- \[ ]  
  
 매칭하고자 하는 캐릭터의 클래스를 가리킨다. 캐릭터는 개인일 수 있고, 범위일 수 도 있다. [abc] 는 a 또는 b 또는 c 가 있는 문자열과 매칭된다. 이것은 [a-c] 와 같은 의미이다. 만약 소문자 영단어와 매칭되길 원하면, [a-z] 를 사용할 수 있다. 
  
  \\ 를 제외한 다른 metacharacter 들은 [ ] 내부에서 의미를 가지지 않는다. [akm$] 는 'a' , 'k', 'm','$' 의 문자 중 하나와 매칭될 것이다. 
  
  [\^5] 는   5를 제외한 문자들과 매칭된다. 하지만 [5^] 는 '5' 또는 '^' 와 매칭될 것이다. 
 



## 자주 사용하는 정규표현식 

| 표현식 | 의미                                         |
| ------ | -------------------------------------------- |
| \^X     | 문자열이 X로 시작됨                          |
| X$     | 문자열이 X로 끝남                            |
| .      | 임의의 한 문자                               |
| X*     | 문자 X가 0번 이상 반복됨                      |
| X+     | 문자 X가 1번 이상 반복됨                      |
| X?     | 문자 X가 존재할 수도, 존재하지 않을 수도 있음 |
| X\|Y   | 문자 X 또는 문자 Y 가 존재                    |
| (X)    | 문자 X를 정규표현식 그룹으로 처리함          |
| X{n}   | 문자X가 n번 반복됨                           |
| X{n,}  | 문자 X가 n번 이상 반복됨                     |
| X{n,m} | 문자 X가 n번 이상 m번 이하 반복됨            |
| [XY]   | 문자X또는 문자Y 중 하나임                    |
| \[^X]  | 문자 X를 제외한 문자                         |
| [A-Z]  | 문자 A부터 문자 Z까지의 문자 (범위)          | 

자주 사용하는 문자 클래스 

| 표기법 | 의미                              |
| ------ | --------------------------------- |
| \d     | 숫자[0-9]                         |
| \D     | 숫자가 아닌 것 \[^0-9]            |
| \s     | 공백(whitespace)문자 [\t\n\r\f\v] |
| \S     | 공백 문자가 아닌 것               |
| \w     | 문자 + 숫자 [a-zA-Z0-9_]          |
| \W     | 문자 + 숫자가 아닌 것             | 

## 정규표현식 모듈 함수

  

| 모듈 함수 | 설명 |
| ---- | ---- |
| `re.compile()` | 정규표현식을 컴파일하는 함수이다. 찾고자 하는 패턴이 빈번한 경우에는 미리 컴파일해놓고 사용하면 속도와 편의성면에서 유리하다. |
| `re.search()` | 문자열 전체에 대해서 정규표현식과 매치되는지를 검색한다. |
| `re.match()` | 문자열의 처음이 정규표현식과 매치되는지를 검색한다. |
| `re.fullmatch()` | 전체 문자열이 일치하는지 확인하고 일치하는 문자 리턴한다.  |
| `re.split()` | 정규 표현식을 기준으로 문자열을 분리하여 리스트로 리턴한다. |
| `re.findall()` | 문자열에서 정규 표현식과 매치되는 모든 경우의 문자열을 찾아서 리스트로 리턴한다. 만약, 매치되는 문자열이 없다면 빈 리스트가 리턴된다. |
| `re.finditer()` | 문자열에서 정규 표현식과 매치되는 모든 경우의 문자열에 대한 이터레이터 객체를 리턴한다. |
| `re.sub()` | 문자열에서 정규 표현식과 일치하는 부분에 대해서 다른 문자열로 대체한다. |


## 실제 예시

### 정규표현식 예시

실제 예시를 통해 더 알아보자.

1. 전화번호 추출
   
```text
Luke Skywalker 02-123-4567 luke@starwars.com
다스베이더 080-8888-8888 darth_vader@gmail.com
leia 010 3434 3221 leia@naver.com
```

위의 텍스트에서 전화번호만 찾는다고 하자. 이때 사용할 정규표현식은 아래와 같다.
```
0\d{1,2}[ -]?\d{3,4}[ -]?\d{3,4}
```

2. pdf 파일에서 파일명 추출

주어진 텍스트
```
file_record_transcript.pdf
regex_101.pdf
```

정규표현식
```
^(.+)\.pdf$
```

추출 결과
`file_record_transcript`, `regex_101`

### re 모듈 사용하기 

```python
text = "Luke Skywalker 02-123-4567 luke@starwars.com, 다스베이더 080-8888-8888 darth_vader@gmail.com, leia 010 3434 3221 leia@naver.com"
r = re.compile("0\d{1,2}[ -]?\d{3,4}[ -]?\d{3,4}")
print(r.findall(text))>)

# 출력 결과
# ['02-123-4567', '080-8888-8888', '010 3434 3221']

```

```python
# 공백 기준 분리
text = "사과 딸기 수박 메론 바나나"
re.split(" ", text)

# ['사과', '딸기', '수박', '메론', '바나나']  
```

```python
text = "Regular expression : A regular expression, regex or regexp[1] (sometimes called a rational expression)[2][3] is, in theoretical computer science and formal language theory, a sequence of characters that define a search pattern."

preprocessed_text = re.sub('[^a-zA-Z]', ' ', text) #특수 문자 공백으로 대체
print(preprocessed_text)

"""
출력결과
'Regular expression   A regular expression  regex or regexp     sometimes called a rational expression        is  in theoretical computer science and formal language theory  a sequence of characters that define a search pattern '  

"""

```


## Reference
- https://docs.python.org/3/library/re.html
- https://docs.python.org/3/howto/regex.html#regex-howto
- https://wikidocs.net/21703