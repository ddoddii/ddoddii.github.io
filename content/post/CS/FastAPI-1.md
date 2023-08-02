+++
author = "Soeun"
title = "FastAPI 기초"
date = "2023-07-27"
description = "HTTP Request Method 설계하기"
categories = [
    "CS"
]
tags = [
    "FastAPI"
]
image = ""
+++

## FastAPI 란?

- Django, FastAPI, Flask 등 웹서비스의 백엔드 프레임워크는 다양하다.
- 그 중에서 왜 FastAPI 에 대해 공부해봤나 ?
  - ASGI(비동기 처리 (async) 가능 = 병렬 처리 가능)
  - pydantic을 이용한 validation -> API 사용자, 개발자 간 소통을 정확하게
  - OpenAPI 문서 자동 생성 -> LocalHost/docs 로 이동하면 FastAPI 에서 만들어놓은 Swagger UI 문서로 테스트 가능 
  - 뛰어난 문서->따라하기 쉽고 상세한 문서가 있다 
    - [FastAPI 공식문서](https://github.com/tiangolo/fastapi)
- 내 목적인 ML Model Deploy를 하기 위해 가장 배우기 쉽고, 따라할 수 있는 문서가 많아서 선택해서 배워보기로 했다. 

## FastAPI 이용해서 API 설계하기

- HTTP Method 에는 POST, GET, PUT, DELETE 가 있다. 이것은 데이터를 처리하는 가장 기본 규칙인 CRUD (Create, Read, Update, Delete) 에 맞게 설계된 규칙이다. 

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0e4b6f86-c7d9-4ec2-9386-b9f6f1402f44)

### GET 

우선 아래와 같이 내 FastAPI 를 만들었다. 
```python 
from fastapi import FastAPI, Body
app = FastAPI(
    title = "API Practice"
    version = "1.0"
    )
```

**1. 단순하게 정보를 읽어오는 GET 을 구현하기**

```python
@app.get('/')
async def first_api():
    return ("FastAPI 를 배워봅시다!")


@app.get('/api-endpoint')
async def first_api_endpoint():
    return ("내 api-endpoint !")
```

@app.get('/api-endpoint') 에서 / 뒤에 있는 것은 내 api-endpoint 이다. 
이제 uvicorn 서버를 실행시켜서 결과물을 확인해보자. 

uvicorn 서버를 실행하려면, 실행하려는 파일이 있는 디렉토리에서 `uvicorn books1:app --reload`를 실행하면 된다. 

여기서 books1 은 내 FastAPI 가 있는 .py 파일 이름이고, app 은 위에 명시한 것 처럼 내 FastAPI 이름이다. 
--reload는 내가 파이썬 파일을 수정하면 바로바로 반영할 수 있게 하는 파라미터이다. 

URL : 127.0.0.1:8000/ 에 접속하면, 아래와 같이 뜰 것이다. 

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/4f670c0f-8631-4c11-85fa-89d3219d0181)

그리고 URL : 127.0.0.1:8000/api-endpoint 에 접속하면, 아래와 같이 뜬다.

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e406555b-5457-4f64-9631-b01cf1f2b3a0)

**2. Dynamic Parameter 사용하기**

내 api endpoint 에 직접 변수를 입력하는 dynamic parameter 를 입력할 수 있다.
아래는 BOOKS 라는 책들의 정보 모음을 저장해두고, book_title 이 내가 가지고 있는 BOOKS 모음에 있으면 그 책에 대한 정보를 리턴하고, 아니면 new_book : book_title 을 리턴하도록 함수를 설계했다. 

```python
BOOKS = [
    {'title': 'Mockingbird', 'author': 'Harper Lee', 'category': 'fiction'},
    {'title': 'MobyDick', 'author': 'Herman Melville', 'category': 'fiction'},
    {'title': '1984', 'author': 'George Owell', 'category': 'history'},
    {'title': 'Great Gatsby', 'author': 'Fitzgerald', 'category': 'fiction'},
    {'title': 'Frankenstien', 'author': 'Shelly', 'category': 'zombie'},
    {'title': 'HarryPotter', 'author': 'J.K.Rolling', 'category': 'fiction'}
]
@app.get('/books/{book_title}')
async def read_book(book_title: str):
    for book in BOOKS:
        if book.get('title').casefold() == book_title.casefold():
            return book
        else:
            return {'new_book': book_title}
```
이렇게 실제 있는 책 정보를 입력하면 그 정보가 뜨고, 

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/d0f5a3cd-f664-4ef8-a2d2-974a170a5323)

없는 책 정보를 입력하면 new_book 을 리턴한다.

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0c0899f5-7da4-42fe-aa60-d6de4177b55a)

**3. Query Parameters**

- {name=value} pair 를 가진 파라미터이다. 
- URL 에서 ?뒤에 붙여진다. 

내가 만약 특정 카테고리에 해당하는 책을 BOOKS 에서 찾고 싶다고 해보자. 
아래 코드는 내가 검색한 카테고리가 BOOKS 내의 카테고리와 일치하면 그 책을 리턴하는 함수이다. 
```python
@app.get("/books/")
async def read_category_by_query(category:str):
    books_to_return = []
    for book in BOOKS:
        if book.get('category').casefold() == category.casefold():
            books_to_return.append(book)
    return books_to_return
```
![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/0a12c646-6a54-44f0-831d-de3587fb99ed)

### POST
- POST 는 새로운 정보를 생성하는 HTTP Method이다. 
- POST 는 Request Body 가 있다. <-> GET 은 Request Body 를 가질 수 없다 

이제 FastAPI 의 큰 장점인 Swagger UI 를 사용해서 내 API 를 테스트 해보자. Swagger UI 접속은 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) 로 이동하면 접속할 수 있다. 

POST Method 를 설계해보자. 나는 새로운 책을 만드는 함수를 만들었다. 

```python
@app.post("/books/create_book")
async def create_book(new_book = Body()):
    BOOKS.append(new_book)
    return BOOKS
```
![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6a3f66bd-822f-46b5-8e17-4e3f81ddd63b)
새로운 책을 추가하면, Status code 200 ok 와 함께 BOOKS 아래 새롭게 추가된 것을 볼 수 있다. 
![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/6363ca78-84be-40f3-abb1-a3e0740ce729)

### PUT
- PUT 은 정보를 업데이트 하는데 사용한다.
  
아래 Body() 는 사용자가 Request Body 에 입력하는 정보이다. 사용자가 입력한 정보를 updated_book 으로 받고, 이 updated_book 의 title 과 BOOKS 내의 title 이 일치하는 것이 있으면 이 updated_book 의 정보로 업데이트 해주는 함수이다. 

조심해야 하는 것이 있는데, Request Body 에서 입력할 때 무조건 쌍따옴표("") 를 사용해야 한다 ! 작은따옴표('')를 입력하면 422 semantic error 가 뜬다. 

```python
@app.put("/books/update_book")
async def update_book(updated_book = Body()):
    for i in range(len(BOOKS)):
        if BOOKS[i].get('title').casefold() == updated_book.get('title').casefold():
            BOOKS[i] = updated_book
    return BOOKS
```

### DELETE
- DELETE 는 있는 정보를 삭제하는 데 사용한다. 

아래 함수는 book_title 을 endpoint 로 입력받고, 입력한 book_tile 과 BOOKS 의 title 이 일치하는 것이 있으면 BOOKS 에서 삭제(pop) 하는 함수이다. 

```python
@app.delete("/books/delete_book/{book_title}")
async def delete_book(book_title:str):
    for i in range(len(BOOKS)):
        if BOOKS[i].get('title').casefold() == book_title.casefold():
            BOOKS.pop(i)
            break
    return BOOKS
```

### Reference 
- Udemy, FastAPI - The complete course 2023

### 사용한 자료와 코드
- 사용한 자료와 코드는 모두 제 GitHub 에서 볼 수 있습니다.
- [GitHub](https://github.com/ddoddii/skills-for-DS/tree/main/week3) `books1.py` 파일 참고