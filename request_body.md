# 요청 본문

클라이언트(예: 브라우저)에서 API로 데이터를 보내야 할 때 요청 본문 으로 보냅니다 .
**요청 본문**은 클라이언트가 API로 보낸 데이터입니다. **응답 본문**은 API가 클라이언트 에 보내는 데이터입니다.
API는 거의 항상 응답 본문을 보내야 합니다. 그러나 클라이언트가 항상 요청 본문 을 보낼 필요는 없습니다.

1. Import Pydantic's BaseModel
    - 먼저 다음에서 가져와야 BaseModel합니다 pydantic.

코드
```
from pydantic import BaseModel
```

## 데이터 모델 만들기

그런 다음 데이터 모델을 에서 상속하는 클래스로 선언합니다 BaseModel.
모든 속성에 대해 표준 Python 유형을 사용합니다.

코드
```
from typing import Union
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
```

## 매개변수로 선언

경로 작업 에 추가하려면 경로 및 쿼리 매개변수를 선언한 것과 같은 방식으로 선언합니다.

코드
```
 ...(생략)...
app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

## 결과

Python 유형 선언 만으로 FastAPI 는 다음을 수행합니다.

- 요청 본문을 JSON으로 읽습니다.
- 해당 유형을 변환합니다(필요한 경우).
- 데이터를 검증합니다.
    - 데이터가 유효하지 않은 경우 정확하고 잘못된 데이터가 무엇인지를 나타내는 명확하고 명확한 오류를 반환합니다.
- 매개변수에 수신된 데이터를 제공합니다 item.
    - 함수에서 type 으로 선언했으므로 Item모든 속성과 해당 유형에 대한 모든 편집기 지원(완성 등)도 갖게 됩니다.
- 모델에 대한 JSON 스키마 정의를 생성 하고 프로젝트에 적합한 경우 원하는 다른 곳에서 사용할 수도 있습니다.
- 이러한 스키마는 생성된 OpenAPI 스키마의 일부이며 자동 문서 UI 에서 사용됩니다 .

## 모델 사용

함수 내부에서 모델 객체의 모든 속성에 직접 액세스할 수 있습니다.

코드
```
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    item_dict = item.dict()
    if item.tax:
        price_with_tax = item.price + item.tax
        item_dict.update({"price_with_tax": price_with_tax})
    return item_dict
```
> 핵심코드 :<br>
if item.tax:<br>
    price_with_tax = item.price + item.tax <br>
    item_dict.update({"price_with_tax": price_with_tax})

## 요청 본문 + 경로 매개변수

경로 매개변수와 요청 본문을 동시에 선언할 수 있습니다.

FastAPI 는 경로 매개변수와 일치하는 함수 매개변수를 경로에서 가져와야 하고 Pydantic 모델로 선언된 함수 매개변수를 요청 본문에서 가져와야 함을 인식합니다.

코드
```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

app = FastAPI()

@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

## 요청 본문 + 경로 + 쿼리 매개변수

또한 body , path 및 쿼리 매개변수를 동시에 선언할 수도 있습니다 .

FastAPI 는 각각을 인식하고 올바른 위치에서 데이터를 가져옵니다.

코드
```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

app = FastAPI()

@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: Union[str, None] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

## Pydantic 모듈 없이 사용하는방법

Pydantic 모델을 사용하지 않으려면 Body 매개변수 를 사용할 수도 있습니다.
[Body - Multiple Parameters: Singular values ​​in body 문서](https://fastapi.tiangolo.com/ko/tutorial/body-multiple-params/#singular-values-in-body)