+++
author = "Soeun"
title = "FastAPI 활용"
date = "2023-07-30"
summary = "Pydantic, HTTPException 사용하기"
categories = [
    "CS"
]
tags = [
    "FastAPI"
]
image = ""
+++

## Pydantics
- POST 에서 정보를 줄 때 validation 을 하고 싶다면? -> Pydantics 이용 !
- 데이터 모델링, 데이터 parsing 을 할 때 에러를 핸들링하는 라이브러리 

```python
from fastapi import FastAPI , Body, Path, Query, HTTPException
from pydantic import BaseModel, Field
from typing import Optional
```
1편에서 했던 BOOKS 정보를 class 로 틀을 만들어보자. 

```python
class Book:
    id : int
    title : str 
    author : str
    description :str
    rating : float
    
    def __init__(self,id,title,author,description,rating):
        self.id = id
        self.title = title
        self.author = author
        self.description = description
        self.rating = rating

BOOKS = [
    Book(1,"Mockingbird","Harper Lee","How to kill a mockingbird",4),
    Book(2,"MobyDick","Herman Melville","Book about whales",3),
    Book(3,"1984","George Owell","Book about distopia",4.5),
    Book(4,"Great Gatsby","Fitzgerald","Gatsby's life story",5),
    Book(5,"Harry Potter","J.K.Rolling","Harry going to Hogwarts",5),
    Book(6,"Frankenstien","Shelly","Zombie book",2)
]
```
만약 내가 POST 로 BOOKS 안에 내용을 추가할 때, BOOK 의 속성인 rating 이 0~5 사이가 되도록 하고 싶다면 ? 
title 정보에는 3글자 이상 입력하도록 하고 싶다면 ? 
author 정보에는 1글자 이상 입력하도록 하고 싶다면 ?

이러한 정보들을 미리 검증해주고, 조건에 안 맞으면 Error 를 리턴해주는 것이 pydantic 의 BaseModel 이다.
아래 BookRequest 에는 pydantic 의 BaseModel 을 상속하여 규칙을 작성하였다.

id : Optional 이다. = 꼭 있지 않아도 된다.
title : string 이며, 최소 길이는 3이다.
author : string 이며, 최소 길이는 1이다.
description : string 이며, 최소 길이는 1, 최대 길이는 100이다.
rating: float 이며, 0보다 크고(greater than) , 6보다 작다(less than).

```python
class BookRequest(BaseModel):
    id: Optional[int] = Field(title='id is not needed')
    title: str = Field(min_length=3)
    author: str = Field(min_length=1)
    description: str = Field(min_length=1, max_length=100)
    rating: float = Field(gt=0, lt=6)
```

이렇게 하고, 아래 POST 함수를 작성했다. 이제 새로 들어온 book_request 는 BookRequest 의 형식에 맞아야 한다. 
`new_book = Book(**book_request.dict())` 는 book_request 를 Book 클래스의 새로운 인스턴스(new_book)으로 만들어주라는 뜻이다. 

```python
@app.post("/create_book")
async def create_book(book_request : BookRequest):
    new_book = Book(**book_request.dict())
    BOOKS.append(find_book_id(new_book))
    return BOOKS=
```

그래서 실제 docs 에서 rating 에 10 을 입력하면 422 error 가 나타나는 것을 확인할 수 있다.

![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/b240eb1d-1d2b-4739-ad5c-4e1153cdee88)
![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/786c8ea1-332d-4129-b170-c30ef0975aeb)

그리고 내가 만약 swagger ui 에 example 을 작성하여 API 사용자들에게 안내하고 싶다면 (이런 습관이 중요하다) 아래와 같이 작성하면 된다. 
```python
class BookRequest(BaseModel):
    id: Optional[int] = Field(title='id is not needed')
    title: str = Field(min_length=3)
    author: str = Field(min_length=1)
    description: str = Field(min_length=1, max_length=100)
    rating: float = Field(gt=0, lt=6)
    
    class Config:
        schema_extra = {
            'example': {
                'title' : 'A new book',
                'author' : 'Put in Author',
                'description' : 'New description of book',
                'rating' : 'Put in rating from 0~6',
            }
        }
```
그러면 Request Body 에 아래와 같이 친절하게 설명이 뜬다. 
![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/e14f6d92-cf81-4e20-9d8a-20f77c11139a)

## Path, Query 이용해 validate 하기 
- Path, Query 를 이용해 검증할 수 도 있다.
- Path : 첫번째 read_book 는 Path 에 있는 book_id 가 0보다 큰 것만 입력을 받도록 했다. 
- Query : 두번째 read_book_by_rating 은 book_rating 이라는 쿼리를 이용해서 book 을 찾는다. 따라서 Query 를 이용해서 book_rating 이 0보다 크고 6보다 작은 값만 들어오도록 했다.

```python
@app.get("/books/{book_id}")
async def read_book(book_id : int =Path(gt=0)):
    for book in BOOKS:
        if book.id == book_id:
            return book

@app.get("/books/")
async def read_book_by_rating(book_rating:float = Query(gt=0,lt=6)):
    books_to_return = []
    for book in BOOKS:
        if book.rating == book_rating:
            books_to_return.append(book)
    return books_to_return
```
## HTTP Exceptions 

### HTTP Response code 

클라이언트와 웹서버가 통신 할 때, 통신이 제대로 되었는지, 혹은 오류가 발생했다면 어디서 발생했는지 알려주도록 정해놓은 상태코드들이 있다. 

대표적인 것들을 알아보자.

- 2xx Sucessful Request
  - 200 OK : 요청이 성공적 !
  - 201 Created : Post 로 새로운 정보 생성 성공
  - 204 No Content :PUT 로 업데이트는 했지만 리턴값이  없을 때
  
- 4xx Client Error Status Code
  - 400 Bad Request : 잘못된 문법으로 서버가 요청을 이해할 수 없다
  - 401 Unauthorized : 클라이언트가 미인증 상태여서 요청을 수행할 수 없다
  - 404 Not Found: 서버가 요청 받은 리소스를 찾을 수 없다
  - 422 Unprocessable Entity : 내가 보낸 정보에 semantic error 가 있다 


- 5xx Internal Server Error
    - 500 Internal Server Error : 서버에서 문제가 발생했다  

이 Status code 를 사용해서 API 사용자에게 어디에서 오류가 났는지 알려줄 수 있다. 
이것의 설계는 HTTPExceptions 를 이용해서 한다. 
상태 코드를 파악하고 상황에 맞는 Error 를 리턴해주자 ! 

아래는 book_id 를 이용해서 book 을 찾고 싶을 때 찾고 싶은 book_id 가 없다면, 404 Not found Error 를 리턴해주는 함수이다. 

```python
@app.get("/books/{book_id}")
async def read_book(book_id : int =Path(gt=0)):
    for book in BOOKS:
        if book.id == book_id:
            return book
    raise HTTPException(status_code=404, detail = "Item not found")
```

이렇게 FastAPI 에서 기초적인 것과 중요한 것들을 다루어 보았다 😊

### Reference
- Udemy, FastAPI - The complete course 2023

### 사용한 자료와 코드
- 사용한 자료와 코드는 모두 제 GiHhub 에서 볼 수 있습니다.
- [GitHub](https://github.com/ddoddii/skills-for-DS/tree/main/week3) `books2.py` 파일 참고