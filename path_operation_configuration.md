# 경로 작업 구성

경로 작업 데코레이터 에 전달하여 구성할 수 있는 몇 가지 매개변수가 있습니다.

## 응답 상태 코드

경로 작업status_code 의 응답에 사용할 (HTTP)를 정의할 수 있습니다 .

int와 같이 코드를 직접 전달할 수 있습니다 404.

그러나 각 숫자 코드의 용도가 기억나지 않는 경우 다음에서 바로 가기 상수를 사용할 수 있습니다 status.

```
from typing import Set, Union
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()

@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```

해당 상태 코드는 응답에 사용되며 OpenAPI 스키마에 추가됩니다.

## 태그

경로 작업 에 태그를 추가 하고 매개변수 tags를 다음과 같이 전달할 수 있습니다(일반적 list으로 str하나만 str).

```
from typing import Set, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()

@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
    return item

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

## 열거형 태그

큰 응용 프로그램이 있는 경우 여러 태그 가 누적될 수 있으며 관련 경로 작업 에 항상 동일한 태그 를 사용하도록 하고 싶을 것 입니다.
이러한 경우 태그를 Enum.
FastAPI 는 일반 문자열과 동일한 방식으로 이를 지원합니다.

```
from enum import Enum
from fastapi import FastAPI

app = FastAPI()

class Tags(Enum):
    items = "items"
    users = "users"

@app.get("/items/", tags=[Tags.items])
async def get_items():
    return ["Portal gun", "Plumbus"]

@app.get("/users/", tags=[Tags.users])
async def read_users():
    return ["Rick", "Morty"]
```

## 요약 및 설명

summary다음 을 추가할 수 있습니다 description.


```
from typing import Set, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()

@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
async def create_item(item: Item):
    return item
```

## docstring의 설명

설명은 길고 여러 줄을 포함하는 경향이 있으므로 함수 docstring 에서 경로 작업 설명을 선언할 수 있으며 FastAPI 는 거기에서 이를 읽습니다.

독스트링에 Markdown 을 작성할 수 있습니다 . 그러면 올바르게 해석되고 표시됩니다(독스트링 들여쓰기를 고려하여).

```
from typing import Set, Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()

@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

## 응답 설명

매개변수를 사용하여 응답 설명을 지정할 수 있습니다 response_description.

```
from typing import Set, Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()

@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    response_description="The created item",
)
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

## 경로 작업 지원 중단

경로 작업 을 deprecated 로 표시해야 하지만 제거하지 않고 다음 매개변수를 전달합니다 deprecated.

```
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]

@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

### 요약

경로 작업 데코레이터 에 매개변수를 전달 하여 경로 작업 에 대한 메타데이터를 쉽게 구성하고 추가할 수 있습니다.