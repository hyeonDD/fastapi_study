# 본문 - 필드

## import field

```
from pydantic import Field
```

와 같이 Field를 임포트 해주어야 합니다.

전체 코드예시
```
from typing import Union
from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: Union[float, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

## 모델 속성 선언

그런 다음 Field모델 속성과 함께 사용할 수 있습니다.

```
from typing import Union
from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str
    description: Union[str, None] = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: Union[float, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

## 추가 정보 추가

Field, Query, 등에 추가 정보를 선언할 수 Body있으며 생성된 JSON 스키마에 포함됩니다.

예제 선언 방법을 배울 때 문서의 뒷부분에서 추가 정보를 추가하는 방법에 대해 자세히 알아볼 것입니다.

### 요약

Pydantic을 사용 Field하여 모델 속성에 대한 추가 유효성 검사 및 메타데이터를 선언할 수 있습니다.

추가 키워드 인수를 사용하여 추가 JSON 스키마 메타데이터를 전달할 수도 있습니다.