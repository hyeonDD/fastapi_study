# 응답 모델

response_model모든 경로 작업 에서 매개변수를 사용하여 응답에 사용되는 모델을 선언할 수 있습니다.

- @app.get()
- @app.post()
- @app.put()
- @app.delete()
- 등.

```
from typing import List, Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: List[str] = []

@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    return item
```
> " decorator response_model" 메소드( get, post, 등)의 매개변수입니다. 모든 매개 변수 및 본문과 같이 경로 작업 기능 이 아닙니다 .

이것은 Pydantic 모델 속성에 대해 선언하는 것과 동일한 유형을 수신하므로 Pydantic 모델일 수 있지만 예를 들어 list와 같은 Pydantic 모델일 수도 있습니다 List[Item].

FastAPI는 이를 사용하여 다음(response_model)을 수행 합니다.

- 출력 데이터를 해당 유형 선언으로 변환합니다.
- 데이터를 검증합니다.
- OpenAPI 경로 작업 에서 응답에 대한 JSON 스키마를 추가합니다 .
- 자동 문서화 시스템에서 사용됩니다.

그러나 가장 중요한 것은:
- 출력 데이터를 모델의 데이터로 제한합니다. 이것이 얼마나 중요한지 아래에서 살펴보겠습니다.
> 응답 모델은 함수 반환 유형 주석 대신 이 매개변수에서 선언됩니다. 경로 함수는 실제로 해당 응답 모델을 반환하지 않고 오히려 dict, 데이터베이스 개체 또는 일부 다른 모델을 반환한 다음 response_model필드 제한 및 직렬화.

## 동일한 입력 데이터 반환

여기에서 UserIn모델을 선언하고 있으며 여기에는 일반 텍스트 암호가 포함됩니다.

```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Union[str, None] = None

# Don't do this in production!
@app.post("/user/", response_model=UserIn)
async def create_user(user: UserIn):
    return user
```

그리고 이 모델을 사용하여 입력을 선언하고 동일한 모델을 사용하여 출력을 선언합니다.

이제 브라우저가 비밀번호로 사용자를 생성할 때마다 API는 응답에서 동일한 비밀번호를 반환합니다.
이 경우 사용자 자신이 암호를 보내는 것이기 때문에 문제가 되지 않을 수 있습니다.
그러나 다른 경로 작업 에 동일한 모델을 사용하는 경우 사용자의 암호를 모든 클라이언트에 보낼 수 있습니다.

> 사용자의 일반 암호를 저장하거나 응답으로 보내지 마십시오.

## 출력 모델 추가

대신 일반 텍스트 암호가 있는 입력 모델과 암호가 없는 출력 모델을 만들 수 있습니다.

```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Union[str, None] = None

class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Union[str, None] = None

@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user
```
> @app.post("/user/", response_model=UserOut) : response_model=UserOut 출력 모델을 UserOut으로 지정함.

## 응답 모델 인코딩 매개변수

응답 모델에는 다음과 같은 기본값이 있을 수 있습니다.

```
from typing import List, Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

- description: Union[str, None] = None의 기본값이 None있습니다.
- tax: float = 10.5의 기본값이 10.5있습니다.
- tags: List[str] = []빈 목록의 기본값: [].

그러나 실제로 저장되지 않은 경우 결과에서 생략할 수 있습니다.

예를 들어 NoSQL 데이터베이스에 많은 선택적 속성이 있는 모델이 있지만 기본값으로 가득 찬 매우 긴 JSON 응답을 보내고 싶지 않은 경우입니다.

## response_model_exclude_unset 매개변수 사용 (null은 출력x)

경로 작업 데코레이터 매개변수 를 설정할 수 있습니다 response_model_exclude_unset=True.

해당 기본값은 응답에 포함되지 않고 실제로 설정된 값만 포함됩니다.

따라서 ID가 있는 항목에 대한 해당 경로 작업 에 요청을 보내는 경우 foo응답(기본값 ​​제외)은 다음과 같습니다.

```
{
    "name": "Foo",
    "price": 50.2
}
```

> 기본값은 만이 아니라 무엇이든 될 수 있습니다 None.<br>목록( ) [], ~ 등일 수 있습니다.float10.5

## response_model_exclude와 response_model_include

response_model_exclude 는 제외,
response_model_include 는 포함이다.

### 요약

경로 작업 데코레이터의 매개변수 를 사용 response_model하여 응답 모델을 정의하고 특히 개인 데이터가 필터링되도록 합니다.

response_model_exclude_unset명시적으로 설정된 값만 반환하는 데 사용 합니다.