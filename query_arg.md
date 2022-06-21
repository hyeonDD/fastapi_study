# 쿼리 매개변수

## 쿼리 매개변수

경로 매개변수의 일부가 아닌 다른 함수 매개변수를 선언할 때, "쿼리" 매개변수로 자동 해석합니다.

코드
```
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```

## 선택적 매개변수

같은 방법으로 기본값을 None으로 설정하여 선택적 매개변수를 선언할 수 있습니다:

코드
```
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```

## 쿼리 매개변수 형변환

bool 형으로 선언할 수도 있고, 아래처럼 변환됩니다:

코드
```
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: Union[str, None] = None, short: bool = False):
    item = {"item_id": item_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

## 여러 경로/쿼리 매개변수

여러 경로 매개변수와 쿼리 매개변수를 동시에 선언할 수 있으며 FastAPI는 어느 것이 무엇인지 알고 있습니다.

- 사용법
    - 그리고 특정 순서로 선언할 필요가 없습니다.
    -  매개변수들은 이름으로 감지됩니다

코드
```
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Union[str, None] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```

## 필수 쿼리 매개변수

- 사용법
    - 경로가 아닌 매개변수에 대한 기본값을 선언할 때(지금은 쿼리 매개변수만 보았습니다), 해당 매개변수는 필수적(Required)이지 않았습니다.
    - 특정값을 추가하지 않고 선택적으로 만들기 위해선 기본값을 None으로 설정하면 됩니다.
    - 그러나 쿼리 매개변수를 필수로 만들려면 기본값을 선언할 수 없습니다

코드
```
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, needy: str, skip: int = 0, limit: Union[int, None] = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item
```

이 경우 3가지 쿼리 매개변수가 있습니다

- needy, 필수적인 str.
- skip, 기본값이 0인 int.
- limit, 선택적인 int.

> 경로 매개변수와 마찬가지로 Enum을 사용할 수 있습니다.