# 본문 - 업데이트

## 다음으로 교체 업데이트PUT

항목을 업데이트하려면 HTTP PUT 작업 을 사용할 수 있습니다.

를 사용하여 jsonable_encoder입력 데이터를 JSON으로 저장할 수 있는 데이터로 변환할 수 있습니다(예: NoSQL 데이터베이스 사용). 예를 들어 로 변환 datetime합니다 str.

```
from typing import List, Union

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.put("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    update_item_encoded = jsonable_encoder(item)
    items[item_id] = update_item_encoded
    return update_item_encoded
```

PUT 기존 데이터를 대체해야 하는 데이터를 수신하는 데 사용됩니다.

## 부분 업데이트PATCH

HTTP PATCH 작업을 사용하여 데이터를 부분적으로 업데이트 할 수도 있습니다 .

즉, 나머지는 그대로 두고 업데이트하려는 데이터만 보낼 수 있습니다.

> PATCH보다 덜 일반적으로 사용 및 알려져 PUT있습니다.<br>그리고 많은 팀 PUT은 부분 업데이트에도 만 사용합니다.<br>원하는 대로 자유롭게 사용할 수 있으며 FastAPI 는 제한을 두지 않습니다.<br>그러나 이 가이드는 그것들이 어떻게 사용되도록 의도되었는지 어느 정도 보여줍니다.

## Pydantic의 exclude_unset매개변수 사용

exclude_unset부분 업데이트를 받고 싶다면 Pydantic의 모델에서 매개변수를 사용하는 것이 매우 유용합니다 .dict().

처럼 item.dict(exclude_unset=True).

그러면 기본값을 제외 dict하고 모델을 생성할 때 설정한 데이터만 생성됩니다.item

그런 다음 이것을 사용하여 dict기본 값을 생략하고 설정된(요청에서 전송된) 데이터만 생성할 수 있습니다.

```
from typing import List, Union
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

## Pydantic의 update매개변수 사용

이제 를 사용하여 기존 모델의 복사본을 만들고 업데이트할 데이터가 포함된 매개변수를 .copy()전달할 수 있습니다.updatedict

좋아요 stored_item_model.copy(update=update_data):

```
from typing import List, Union
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

## 부분 업데이트 요약

요약하면 부분 업데이트를 적용하려면 다음을 수행합니다.

- (선택 사항) PATCH대신 PUT.
- 저장된 데이터를 검색합니다.
- 해당 데이터를 Pydantic 모델에 넣습니다.
- dict를 사용하여 입력 모델에서 기본값 없이 a를 생성합니다 exclude_unset.
    - 이렇게 하면 이미 모델에 기본값으로 저장된 값을 재정의하는 대신 사용자가 실제로 설정한 값만 업데이트할 수 있습니다.
- 저장된 모델의 복사본을 만들고 수신된 부분 업데이트로 속성을 업데이트합니다( update매개변수 사용).
- 복사된 모델을 DB에 저장할 수 있는 것으로 변환합니다(예: jsonable_encoder).
    - 이는 모델의 .dict()메서드를 다시 사용하는 것과 비슷하지만 값을 JSON으로 변환할 수 있는 데이터 유형(예 datetime: str.
- 데이터를 DB에 저장합니다.
- 업데이트된 모델을 반환합니다.

```
from typing import List, Union
from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: Union[str, None] = None
    description: Union[str, None] = None
    price: Union[float, None] = None
    tax: float = 10.5
    tags: List[str] = []

items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.get("/items/{item_id}", response_model=Item)
async def read_item(item_id: str):
    return items[item_id]

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```

> 입력 모델은 여전히 ​​유효성이 검사됩니다.<br>따라서 모든 속성을 생략할 수 있는 부분 업데이트를 수신하려면 모든 속성이 선택적(기본값 또는 None)으로 표시된 모델이 있어야 합니다.<br>업데이트 에 대한 모든 선택적 값이 있는 모델과 생성 에 필요한 값이 있는 모델을 구별하려면 추가 모델 에 설명된 아이디어를 사용할 수 있습니다 .