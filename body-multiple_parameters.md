# 본문 - 여러 매개변수

Path이제 및 를 사용하는 방법을 보았 Query으므로 요청 본문 선언의 고급 사용법을 살펴보겠습니다.

## 믹스 Path및 Query바디 매개변수

첫째, 물론, 당신은 자유롭게 몸 매개변수 선언을 혼합하고 요청할 수 있으며 FastAPI는 Path,Query가 무엇을 해야 하는지 알 것입니다.

None또한 기본값을 다음 으로 설정하여 본문 매개변수를 선택사항으로 선언할 수도 있습니다.

```
from typing import Union
from fastapi import FastAPI, Path
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000),
    q: Union[str, None] = None,
    item: Union[Item, None] = None,
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    if item:
        results.update({"item": item})
    return results
```

## 다중 본체 매개변수

```
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

class User(BaseModel):
    username: str
    full_name: Union[str, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

이 경우 FastAPI 는 함수에 둘 이상의 본문 매개변수(Pydantic 모델인 두 개의 매개변수)가 있음을 알아차립니다.

## 본문의 특이값

와 같은 방식으로 쿼리 Query및 Path경로 매개변수에 대한 추가 데이터를 정의하기 위해 FastAPI 는 동등한 Body.

예를 들어, 이전 모델을 확장하여 동일한 본문에 및 importance외에 다른 키를 갖도록 결정할 수 있습니다 .itemuser

단수 값이기 때문에 그대로 선언하면 FastAPI 는 쿼리 매개 변수로 가정합니다.

그러나 다음을 사용하여 다른 본문 키로 처리 하도록 FastAPIBody 에 지시할 수 있습니다 .

```
from typing import Union

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

class User(BaseModel):
    username: str
    full_name: Union[str, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User, importance: int = Body()):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

## 여러 본문 매개변수 및 쿼리

물론 필요할 때마다 본문 매개변수 외에 추가 쿼리 매개변수를 선언할 수도 있습니다.

기본적으로 단일 값은 쿼리 매개변수로 해석되므로 명시적으로 a Query를 추가할 필요가 없습니다.

예시 코드
```
from typing import Union

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

class User(BaseModel):
    username: str
    full_name: Union[str, None] = None

@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Item,
    user: User,
    importance: int = Body(gt=0),
    q: Union[str, None] = None
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    if q:
        results.update({"q": q})
    return results
```

## 단일 본문 매개변수 포함

itemPydantic 모델의 바디 매개변수가 하나만 있다고 가정해 보겠습니다 Item.

기본적으로 FastAPI 는 본문을 직접 예상합니다.

그러나 item추가 본문 매개변수를 선언할 때와 같이 키와 그 내부에 모델 내용이 있는 JSON을 예상하려면 특수 Body매개변수 를 사용할 수 있습니다 embed.

```
item: Item = Body(embed=True)
```

코드
```
from typing import Union
from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

이렇게 작성할시 request body는
```
{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2
}
```
이 아니라
```
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```
와 같이 item: 처럼 item(키)을 명시해줘야한다. 

### 요약

요청이 단일 본문만 가질 수 있는 경우에도 경로 작업 함수 에 여러 본문 매개변수를 추가할 수 있습니다 .

그러나 FastAPI 는 이를 처리하고 함수에 올바른 데이터를 제공하며 경로 작업 에서 올바른 스키마의 유효성을 검사하고 문서화합니다 .

본문의 일부로 수신할 특이값을 선언할 수도 있습니다.

그리고 선언된 매개변수가 하나뿐인 경우에도 키에 본문을 포함 하도록 FastAPI 에 지시할 수 있습니다.