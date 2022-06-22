# 요청 예시 데이터 선언

앱이 수신할 수 있는 데이터의 예를 선언할 수 있습니다.

다음은 몇 가지 방법입니다.

## Pydantic schema_extra

[Pydantic 문서](https://pydantic-docs.helpmanual.io/usage/schema/#schema-customization)

스키마 사용자 정의 에 설명된 대로 및 example를 사용하여 Pydantic 모델에 대해 을 선언할 수 있습니다 .Configschema_extra

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

    class Config:
        schema_extra = {
            "example": {
                "name": "Foo",
                "description": "A very nice Item",
                "price": 35.4,
                "tax": 3.2,
            }
        }

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

> 동일한 기술을 사용하여 JSON 스키마를 확장하고 사용자 지정 추가 정보를 추가할 수 있습니다.<br>예를 들어 프론트엔드 사용자 인터페이스 등에 대한 메타데이터를 추가하는 데 사용할 수 있습니다.

## Field추가 인수

Pydantic 모델과 함께 사용할 때 다른 임의의 인수를 함수에 전달 하여 JSON 스키마Field() 에 대한 추가 정보를 선언할 수도 있습니다 .

이것을 사용하여 example각 필드에 추가할 수 있습니다.

```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(example="Foo")
    description: Union[str, None] = Field(default=None, example="A very nice Item")
    price: float = Field(example=35.4)
    tax: Union[float, None] = Field(default=None, example=3.2)

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```
## example그리고 examplesOpenAPI에서

다음 중 하나를 사용할 때:

- Path()
- Query()
- Header()
- Cookie()
- Body()
- Form()
- File()

OpenAPI 에 추가될 추가 정보가 포함된 데이터 example또는 그룹을 선언할 수도 있습니다 .examples

## Body~와 함께example

다음 example에서 예상되는 데이터 를 전달합니다 Body().

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
async def update_item(
    item_id: int,
    item: Item = Body(
        example={
            "name": "Foo",
            "description": "A very nice Item",
            "price": 35.4,
            "tax": 3.2,
        },
    ),
):
    results = {"item_id": item_id, "item": item}
    return results
```

## Body여러examples

단일 대신 여러 예제 를 사용하여 전달할 example수 있습니다 . 각 예제에는 OpenAPI 에도 추가될 추가 정보가 있습니다 .
examplesdict의 키는 dict각 예를 식별하고 각 값은 다른 dict의 각 특정 예 dict에는 다음 examples이 포함될 수 있습니다.

- summary: 예제에 대한 간단한 설명입니다.
- description: 마크다운 텍스트를 포함할 수 있는 긴 설명입니다.
- value: 이것은 표시된 실제 예입니다(예: dict.
- externalValue: 대체 value, 예제를 가리키는 URL. 이것은 처럼 많은 도구에서 지원되지 않을 수 있습니다 value.

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
async def update_item(
    *,
    item_id: int,
    item: Item = Body(
        examples={
            "normal": {
                "summary": "A normal example",
                "description": "A **normal** item works correctly.",
                "value": {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                },
            },
            "converted": {
                "summary": "An example with converted data",
                "description": "FastAPI can convert price `strings` to actual `numbers` automatically",
                "value": {
                    "name": "Bar",
                    "price": "35.4",
                },
            },
            "invalid": {
                "summary": "Invalid data is rejected with an error",
                "value": {
                    "name": "Baz",
                    "price": "thirty five point four",
                },
            },
        },
    ),
):
    results = {"item_id": item_id, "item": item}
    return results
```