+++
author = "Soeun"
title = "FastAPI Best Practice 로 리팩토링 하기"
date = "2024-02-10"
summary = "FastAPI Best Practice"
categories = [
    "CS"
]
tags = [
    "FastAPI",
]
slug = "best-practice"
+++

FastAPI 는 Spring, Django 와는 다르게 개발자의 입맛에 커스텀하기 좋은, 자유도가 높은 프레임워크 입니다. 스프링과 장고는 어느 개발자가 구현해도 뼈대가 비슷하지만, FastAPI 는 자유도가 너무 높아 Best-Practice 를 정해놓지 않으면 구조가 산으로 가는 일이 발생했습니다. 따라서, 본 포스팅에서느 제가 프로젝트를 리팩토링하면서 참고했던 [FastAPI Best Practice](https://github.com/zhanymkanov/fastapi-best-practices) 의 원칙들을 소개해보고자 합니다.

## 1. Project Structure. Consistent & predictable

프로젝트 구조는 예측 가능해야 합니다. 예측 가능한 위치에 예측 가능한 코드가 있도록 리팩토링 했습니다. Best-Practice 에서 추천하는 구조 예시는 아래와 같습니다.

```text
fastapi-project
├── alembic/
├── src
│   ├── auth
│   │   ├── router.py
│   │   ├── schemas.py  # pydantic models
│   │   ├── models.py  # db models
│   │   ├── dependencies.py
│   │   ├── config.py  # local configs
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── aws
│   │   ├── client.py  # client model for external service communication
│   │   ├── schemas.py
│   │   ├── config.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   └── utils.py
│   └── posts
│   │   ├── router.py
│   │   ├── schemas.py
│   │   ├── models.py
│   │   ├── dependencies.py
│   │   ├── constants.py
│   │   ├── exceptions.py
│   │   ├── service.py
│   │   └── utils.py
│   ├── config.py  # global configs
│   ├── models.py  # global models
│   ├── exceptions.py  # global exceptions
│   ├── pagination.py  # global module e.g. pagination
│   ├── database.py  # db connection related stuff
│   └── main.py
├── tests/
│   ├── auth
│   ├── aws
│   └── posts
├── templates/
│   └── index.html
├── requirements
│   ├── base.txt
│   ├── dev.txt
│   └── prod.txt
├── .env
├── .gitignore
├── logging.ini
└── alembic.ini
```

1. 모든 도메인 디렉토리들을 `src` 폴더 안에 저장하세요.
   - `scr/` - app 의 가장 상위 디렉토리로, models, configs, constants 를 포함합니다.
   - `scr/main.py` - 프로젝트의 루트로, FastAPI app 을 시작합니다.
2. 각 패키지는 각자의 router, schemas, models 가 있습니다.
   - `router.py` - 각 모듈의 코어로, endpoints 를 포함
   - `schemas.py` - pydantic models 를 포함
   - `services.py` - 비즈니스 로직에 관련된 모듈
   - `dependencies.py` - router dependencies
   - `constants.py` - 모듈과 관련된 상수와 에러 코드
   - `config.py` - env 변수들
   - `utils.py` - 비즈니스로직 함수가 아닌 것들
   - `exceptions.py` - 모듈과 관련된 예외들, e.g. `PostNotFound`, `InvalidUserData`
3. 패키지가 다른 패키지의 서비스, 상수에 의존한다면 - 명확한 모듈 이름과 함께 import 하세요.

```python
from src.auth import constants as auth_constants
from src.notifications import service as notification_service
from src.posts.constants import ErrorCode as PostsErrorCode  # in case we have Standard ErrorCode in constants module of each package
```

### 2. Excessively use Pydantic for data validation

Pydantic 은 데이터를 검증하고 변환하는데 필요한 기능들을 포함하고 있습니다. 요구되는 필드 정의와 디폴트 값 정의 말고도, Pydantic 은 regex, enums, 길이 검증, 이메일 검증 등 다양한 기능들이 있습니다.

```python
from enum import Enum
from pydantic import AnyUrl, BaseModel, EmailStr, Field, constr

class MusicBand(str, Enum):
   AEROSMITH = "AEROSMITH"
   QUEEN = "QUEEN"
   ACDC = "AC/DC"


class UserBase(BaseModel):
    first_name: str = Field(min_length=1, max_length=128)
    username: constr(regex="^[A-Za-z0-9-_]+$", to_lower=True, strip_whitespace=True)
    email: EmailStr
    age: int = Field(ge=18, default=None)  # must be greater or equal to 18
    favorite_band: MusicBand = None  # only "AEROSMITH", "QUEEN", "AC/DC" values are allowed to be inputted
    website: AnyUrl = None

```

하지만 HyperConnect 팀에서 작성한 [고성능 ML 백엔드를 위한 10가지 Python 성능 최적화 팁](https://hyperconnect.github.io/2023/05/30/Python-Performance-Tips.html#5-pydantic%EC%9D%80-%EC%95%84%EC%A3%BC-%EB%8A%90%EB%A6%AC%EB%8B%A4-%EB%B6%88%ED%95%84%EC%9A%94%ED%95%9C-%EA%B3%B3%EC%97%90%EC%84%9C-%EA%B0%80%EA%B8%89%EC%A0%81-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EB%A7%90%EC%9E%90) 의 #5번에서 Pydantic 을 아주 느리며, 불필요한 곳에서 가급적이면 사용하지 말라고 언급했습니다.

이 당시에는 Pydantic v1.0 을 사용해서 속도가 매우 느렸는데, 최근에 Pydantic v2.0 이 나와서 속도 문제가 많이 개선된 것으로 알고 있습니다. Pydantic 1 과 2의 성능을 비교한 글을 첨부하겠습니다. [Pydantic 1 vs 2: A speed comparison](https://janhendrikewers.uk/pydantic-1-vs-2-a-benchmark-test.html)

데이터 검증을 위해 너무 편리한 기능들을 많이 제공해주어서, 엄청난 트래픽이 몰리는 것이 아닌 서버에서는 굉장히 유용하게 사용하고 있습니다.

### 3. Use dependencies for data validation vs DB

Pydantic 은 클라이언트 인풋으로 오는 값들마나 검증할 수 있습니다. 따라서 dependencies 를 활용하여 데이터베이스 제약조건 (e.g. 중복 이메일) 을 검증하는데 사용하세요. 만약 dependencies 를 사용하지 않는다면, 매 endpoint 마다 post_id 검증을 작성해야 하는데, 굉장히 번거롭습니다.

```python
# dependencies.py
async def valid_post_id(post_id: UUID4) -> Mapping:
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()

    return post


# router.py
@router.get("/posts/{post_id}", response_model=PostResponse)
async def get_post_by_id(post: Mapping = Depends(valid_post_id)):
    return post


@router.put("/posts/{post_id}", response_model=PostResponse)
async def update_post(
    update_data: PostUpdate,
    post: Mapping = Depends(valid_post_id),
):
    updated_post: Mapping = await service.update(id=post["id"], data=update_data)
    return updated_post


@router.get("/posts/{post_id}/reviews", response_model=list[ReviewsResponse])
async def get_post_reviews(post: Mapping = Depends(valid_post_id)):
    post_reviews: list[Mapping] = await reviews_service.get_by_post_id(post["id"])
    return post_reviews
```

### 4. Chain dependencies

Dependencies 는 다른 dependencies 를 사용할 수 있습니다.

```python
# dependencies.py
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

async def valid_post_id(post_id: UUID4) -> Mapping:
    post = await service.get_by_id(post_id)
    if not post:
        raise PostNotFound()

    return post


async def parse_jwt_data(
    token: str = Depends(OAuth2PasswordBearer(tokenUrl="/auth/token"))
) -> dict:
    try:
        payload = jwt.decode(token, "JWT_SECRET", algorithms=["HS256"])
    except JWTError:
        raise InvalidCredentials()

    return {"user_id": payload["id"]}


async def valid_owned_post(
    post: Mapping = Depends(valid_post_id),
    token_data: dict = Depends(parse_jwt_data),
) -> Mapping:
    if post["creator_id"] != token_data["user_id"]:
        raise UserNotOwner()

    return post

# router.py
@router.get("/users/{user_id}/posts/{post_id}", response_model=PostResponse)
async def get_user_post(post: Mapping = Depends(valid_owned_post)):
    return post

```

### 5. Decouple & Reuse dependencies. Dependency calls are cached.

Dependencies 는 여러번 재사용될 수 있는데, 호출할 때마다 연산되지 않습니다. FastAPI 는 dependency 연산의 결과를 캐시해둡니다. 예를 들어, `get_post_by_id` 를 호출하는 dependency 가 있다면, 매번 호출할 때마다 DB 를 방문하지 않습니다.

이것을 알면, dependencies 를 작은 여러 개의 함수들로 나눌 수 있습니다. 이 작은 함수들은 다른 dependency 에 포함되어 여러번 재사용할 수 있습니다.

### 6. Follow the REST

RESTful API를 개발하면 다음과 같은 경로에서 dependency를 더 쉽게 재사용할 수 있습니다.

- `GET /courses/:course_id`
- `GET /courses/:course_id/chapters/:chapter_id/lessons`
- `GET /chapters/:chapter_id`

주의해야 할 점은, 경로에서 같은 변수명을 사용하는 것입니다.

### 7.Don't make your routes async, if you have only blocking I/O operations

FastAPI 는 자체적으로 비동기와 동기적 I/O 연산을 효율적으로 처리할 수 있습니다.

- FastAPI 는 `sync` 라우트들을 쓰레드풀에서 실행하고, 블로킹 I/O 연산은 event loop 가 task 를 실행하는 것을 막지 않습니다.
- 라우트가 `async` 로 정의되어 있으면, `await` 에 의해 주기적으로 호출되고, FastAPI 는 사용자가 논블로킹 I/O 연산을 할 것이라고 생각합니다.

주의할 점은, 사용자가 async route 에 블로킹 연산을 하면, 이벤트 루프는 블로킹 연산이 끝나기 전까지 다음 태스크를 실행하지 못하고 기다려야 합니다. 아래의 코드를 봅시다.

```python
import asyncio
import time

@router.get("/terrible-ping")
async def terrible_catastrophic_ping():
    time.sleep(10) # I/O blocking operation for 10 seconds
    pong = service.get_pong()  # I/O blocking operation to get pong from DB

    return {"pong": pong}

@router.get("/good-ping")
def good_ping():
    time.sleep(10) # I/O blocking operation for 10 seconds, but in another thread
    pong = service.get_pong()  # I/O blocking operation to get pong from DB, but in another thread

    return {"pong": pong}

@router.get("/perfect-ping")
async def perfect_ping():
    await asyncio.sleep(10) # non-blocking I/O operation
    pong = await service.async_get_pong()  # non-blocking I/O db call

    return {"pong": pong}
```

1. `GET /terrible-ping` 을 호출할 때

   1. FastAPI 서버는 요청을 받고 핸들링하기 시작합니다.
   2. 서버의 이벤트 루프와 큐에 있는 모든 태스크들은 `time.sleep()` 이 끝날때까지 기다립니다.

      - 서버는 `time.sleep()`이 I/O 태스크가 아니라고 생각하고, 끝날 때까지 기다립니다.
      - 서버는 기다리는 동안 다른 요청을 받지 않습니다.

   3. 서버의 이벤트루프와 큐에 있는 태스크들은 `service.get_pong()` 이 끝날 때까지 기다립니다.

      - 서버는 `service.get_pong()`이 I/O 태스크가 아니라고 생각하고, 끝날 때까지 기다립니다.
      - 서버는 기다리는 동안 다른 요청을 받지 않습니다.

   4. 서버는 응답을 반환합니다. 응답이 끝난 후, 서버는 새로운 요청들을 받기 시작합니다.

2. `GET /good-ping` 을 호출할 때

   1. FastAPI 서버는 요청을 받고 핸들링하기 시작합니다.
   2. FastAPI 는 `good-ping` 을 쓰레드풀로 보내고, worker thread 가 function 을 실행합니다.
   3. `good-ping` 이 실행되는 동안, 이벤트 루프는 큐에서 다음 태스크를 고르고(e.g. 새로운 요청 받기, DB 호출하기), 그들의 작업을 처리합니다.
      - 메인 쓰레드(FastAPI app) 와 별개로, worker thread 는 `time.sleep` 가 끝나길 기다리고, 그 후 `service.get_pong` 이 끝나길 기다립니다.
      - 동기적 연산은 사이드 쓰레드만 블로킹하고, 메인 쓰레드는 블로킹하지 않습니다.
   4. `good-ping` 이 작업을 완료하면, 서버는 결과를 클라이언트에게 반환합니다.

3. `GET /perfect-ping` 을 호출할 때
   1. FastAPI 서버는 요청을 받고 핸들링하기 시작합니다.
   2. FastAPI 는 `asyncio.sleep(10)` 을 await 합니다.
   3. 이벤트 루프는 큐에서 다음 태스크를 고르고(e.g. 새로운 요청 받기, DB 호출하기), 그들의 작업을 처리합니다.
   4. `asyncio.sleep(10)`이 끝나면, 서버는 다음 줄로 가서 `await service.async_get_pong()` 를 await 합니다.
   5. 이벤트 루프는 다음 태스크를 고르고, 그들의 작업을 처리합니다.
   6. `service.async_get_pong()`가 끝나면, 서버는 결과를 클라이언트에게 반환합니다.

여기서 주의할 점은, 논블로킹 awaitable 연산들 또는 쓰레드풀로 보내지는 연산들은 모두 I/O 집중 태스크(e.g. 파일 열기, DB 호출, 외부 API 호출)이어야 합니다.

- CPU 집약적인 작업(e.g. 많은 연산, 데이터 처리, 비디오 인코딩) 을 await 하는 것은 쓸모 없습니다. 왜냐하면 CPU 는 그 작업들을 끝내기 위해 일을 해야 하지만, I/O 연산은 그것을 끝내기 위해 CPU 가 연산을 하지 않고 기다리므로 다른 요청을 처리할 수 있습니다.
- CPU 집약적인 작업을 다른 쓰레드에서 실행하는 것도 소용이 없는데, 왜냐하면 파이썬 GIL이 하나의 쓰레드만 실행하게 제한하기 때문입니다.
- CPU 집약적인 작업을 최적화하고 싶다면, 다른 프로세스에 있는 워커들로 보내야 합니다.

### 8. Custom base model from day 0.

Pydantic의 Base Model 을 사용하면 필요한 모델을 커스터마이즈할 수 있습니다. 아래와 같이 datetime 모델 형태를 커스텀할 수 있습니다.

```python
from datetime import datetime
from typing import Any
from zoneinfo import ZoneInfo

from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel, ConfigDict, model_validator


def convert_datetime_to_gmt(dt: datetime) -> str:
    if not dt.tzinfo:
        dt = dt.replace(tzinfo=ZoneInfo("UTC"))

    return dt.strftime("%Y-%m-%dT%H:%M:%S%z")


class CustomModel(BaseModel):
    model_config = ConfigDict(
        json_encoders={datetime: convert_datetime_to_gmt},
        populate_by_name=True,
    )

    @model_validator(mode="before")
    @classmethod
    def set_null_microseconds(cls, data: dict[str, Any]) -> dict[str, Any]:
        datetime_fields = {
            k: v.replace(microsecond=0)
            for k, v in data.items()
            if isinstance(k, datetime)
        }

        return {**data, **datetime_fields}

    def serializable_dict(self, **kwargs):
        """Return a dict which contains only serializable fields."""
        default_dict = self.model_dump()

        return jsonable_encoder(default_dict)
```

### 9. Docs

FastAPI 에게 더욱 읽기 쉬운 docs 를 만들기 위해 아래 정보들을 넣을 수 있습니다.

- `response_model`, `status_code`, `description`
- 모델과 status 가 변한다면, `responses` 라우트를 사용해 다른 response 별로 doc 를 만들 수 있습니다.

```python
from fastapi import APIRouter, status

router = APIRouter()

@router.post(
    "/endpoints",
    response_model=DefaultResponseModel,  # default response pydantic model
    status_code=status.HTTP_201_CREATED,  # default status code
    description="Description of the well documented endpoint",
    tags=["Endpoint Category"],
    summary="Summary of the Endpoint",
    responses={
        status.HTTP_200_OK: {
            "model": OkResponse, # custom pydantic model for 200 response
            "description": "Ok Response",
        },
        status.HTTP_201_CREATED: {
            "model": CreatedResponse,  # custom pydantic model for 201 response
            "description": "Creates something from user request ",
        },
        status.HTTP_202_ACCEPTED: {
            "model": AcceptedResponse,  # custom pydantic model for 202 response
            "description": "Accepts request and handles it later",
        },
    },
)
async def documented_route():
    pass
```

### 10. Use Pydantic's BaseSettings for configs

Pydantic 의 [pydantic-settings](https://docs.pydantic.dev/latest/concepts/pydantic_settings/) 를 사용하면 환경 변수를 파싱할 수 있습니다.

```python
from pydantic import AnyUrl, PostgresDsn
from pydantic_settings import BaseSettings  # pydantic v2

class AppSettings(BaseSettings):
    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"
        env_prefix = "app_"

    DATABASE_URL: PostgresDsn
    IS_GOOD_ENV: bool = True
    ALLOWED_CORS_ORIGINS: set[AnyUrl]
```

### 11. Migrations. Alembic.

1. 마이그레이션은 정적이며 되돌릴 수 있어야 합니다. 마이그레이션이 동적으로 생성된 데이터에 의존하는 경우 동적인 유일한 것은 데이터의 구조가 아니라 데이터 자체인지 확인하세요.
2. 마이그레이션 이름에는 설명적인 이름과 슬러그를 사용하세요. 슬러그는 변화를 설명해주어야 합니다.
3. 새 마이그레이션을 위해 사람이 읽을 수 있는 파일 템플릿을 설정합니다. 여기서 `date_slug.py` pattern, e.g. `2022-08-24_post_content_idx.py` 를 추천합니다.

```text
# alembic.ini
file_template = %%(year)d-%%(month).2d-%%(day).2d_%%(slug)s
```

### 12. Set tests client async from day 0

DB를 사용하는 테스트코드는 미래에 이벤트 루프 에러를 발생시킬 수 있습니다. 테스트 클라이언트는 비동기로 설정하세요. e.g. [async_asgi_testclient](https://github.com/vinissimus/async-asgi-testclient) or [httpx](https://github.com/encode/starlette/issues/652)

```python
import pytest
from async_asgi_testclient import TestClient

from src.main import app  # inited FastAPI app


@pytest.fixture
async def client():
    host, port = "127.0.0.1", "5555"
    scope = {"client": (host, port)}

    async with TestClient(
        app, scope=scope, headers={"X-User-Fingerprint": "Test"}
    ) as client:
        yield client


@pytest.mark.asyncio
async def test_create_post(client: TestClient):
    resp = await client.post("/posts")

    assert resp.status_code == 201
```

### 13. BackgroundTasks > asyncio.create_task

BackgroundTask 를 사용하면 FastAPI 가 블로킹 루트들을 핸들링하는 것처럼(`sync` 태스크는 쓰레드풀에서, `async` 태스크는 awaited 됨) 블로킹 I/O 와 논블로킹 I/O 모두 효과적으로 실행할 수 있습니다.

여기서 주의할 점은,

- worker 에게 블로킹 I/O 연산을 `async` 라고 표시하지 마세요.
- CPU 집약적인 태스크를 위해 쓰지 마세요.

```python
from fastapi import APIRouter, BackgroundTasks
from pydantic import UUID4

from src.notifications import service as notifications_service


router = APIRouter()


@router.post("/users/{user_id}/email")
async def send_user_email(worker: BackgroundTasks, user_id: UUID4):
    """Send email to user"""
    worker.add_task(notifications_service.send_email, user_id)  # send email after responding client
    return {"status": "ok"}
```
