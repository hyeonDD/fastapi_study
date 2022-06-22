# 본체 - 내포된 모델

FastAPI 를 사용하면 Pydantic 덕분에 임의로 깊이 중첩된 모델을 정의, 검증, 문서화 및 사용할 수 있습니다.

## 목록 필드

속성을 하위 유형으로 정의할 수 있습니다. 예를 들어 Python list:

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
    tags: list = []

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

그러면 tags항목 목록이 만들어집니다. 각 항목의 유형을 선언하지는 않지만.

## 타이핑 가져오기List

Python 3.9 이상에서는 표준을 사용하여 list아래에서 볼 수 있는 것처럼 이러한 유형 주석을 선언할 수 있습니다.

그러나 3.9(3.6 이상) 이전의 Python 버전에서는 먼저 표준 Python typing모듈 에서 가져와야 합니다.

```
from typing import List, Union
```

## list유형 매개변수로 선언

list, dict, 와 같이 유형 매개변수(내부 유형)가 있는 유형을 선언하려면 다음을 수행합니다 tuple.

typingPython 버전 3.9 미만인 경우 모듈 에서 해당 버전을 가져옵니다.
대괄호를 사용하여 내부 유형을 "유형 매개변수"로 전달합니다 [.]
Python 3.9에서는 다음과 같습니다.
```
my_list: list[str]
```
3.9 이전의 Python 버전에서는 다음과 같습니다.
```
from typing import List
my_list: List[str]
```

이것이 유형 선언에 대한 모든 표준 Python 구문입니다.<br>
내부 유형이 있는 모델 속성에 대해 동일한 표준 구문을 사용하십시오.<br>
tags따라서 이 예에서는 구체적으로 "문자열 목록" 을 만들 수 있습니다.

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

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

## 유형 설정

그러나 우리는 그것에 대해 생각하고 태그가 반복되어서는 안 된다는 것을 깨닫습니다. 아마도 고유한 문자열일 것입니다.

그리고 Python에는 고유한 항목 집합에 대한 특수 데이터 유형인 set.

tags그런 다음 문자열 집합으로 선언할 수 있습니다 .

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

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

이렇게 하면 중복 데이터가 포함된 요청을 수신하더라도 고유한 항목 집합으로 변환됩니다.<br>
그리고 해당 데이터를 출력할 때마다 소스에 중복 항목이 있더라도 고유한 항목 집합으로 출력됩니다.<br>
그리고 그에 따라 주석을 달거나 문서화할 것입니다.

## 내포된 모델

Pydantic 모델의 각 속성에는 유형이 있습니다.<br>
그러나 그 유형 자체가 또 다른 Pydantic 모델이 될 수 있습니다.<br>
따라서 특정 속성 이름, 유형 및 유효성 검사를 사용하여 깊게 중첩된 JSON "객체"를 선언할 수 있습니다.<br>
그 모든 것이 임의로 중첩됩니다.

## 하위 모델 정의

예를 들어 다음과 같이 Image모델을 정의할 수 있습니다.

```
from typing import Set, Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    image: Union[Image, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

## 하위 모델을 유형으로 사용

그런 다음 속성 유형으로 사용할 수 있습니다.

```
from typing import Set, Union
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Image(BaseModel):
    url: str
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    image: Union[Image, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

## 다시 말하지만, 해당 선언만 하면 FastAPI 를 사용하여 다음을 얻을 수 있습니다.

중첩 모델의 경우에도 편집기 지원(완성 등)
- 데이터 변환
- 데이터 유효성 검사
- 자동 문서화

## 특수 유형 및 유효성 검사

str, int, 등과 같은 일반 단수 유형과는 별도로 에서 float상속되는 더 복잡한 단수 유형을 사용할 수 있습니다 str.

가지고 있는 모든 옵션을 보려면 Pydantic의 이국적인 유형 에 대한 문서를 확인하세요 . 다음 장에서 몇 가지 예를 볼 수 있습니다.

예를 들어, Image모델에서 필드가 있는 것처럼 Pydantic의 url필드 대신 다음으로 선언할 수 있습니다 .strHttpUrl

```
from typing import Set, Union
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    image: Union[Image, None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```

문자열은 유효한 URL인지 확인하고 JSON 스키마/OpenAPI에 문서화합니다.

## 하위 모델 목록이 있는 속성

Pydantic 모델을 , 등의 하위 유형으로 사용할 수도 list있습니다 set.

```
from typing import List, Set, Union
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    images: Union[List[Image], None] = None

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```
> images이제 키에 이미지 개체 목록이 어떻게 생겼는지 확인 하세요.

## 깊게 내포된 모델

임의로 깊이 중첩된 모델을 정의할 수 있습니다.

```
from typing import List, Set, Union
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None
    tags: Set[str] = set()
    images: Union[List[Image], None] = None

class Offer(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    items: List[Item]

@app.post("/offers/")
async def create_offer(offer: Offer):
    return offer
```
> Offers 의 목록이 있고 , 차례로 s Item의 선택적 목록이 있는 방법에 주목 하세요.Image

## 순수 목록의 본문

예상하는 JSON 본문의 최상위 값이 JSON array( Python list)인 경우 Pydantic 모델에서와 같이 함수의 매개변수에서 유형을 선언할 수 있습니다.

```
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel, HttpUrl

app = FastAPI()

class Image(BaseModel):
    url: HttpUrl
    name: str

@app.post("/images/multiple/")
async def create_multiple_images(images: List[Image]):
    return images
```

## 임의 dict의 몸체

dict또한 일부 유형의 키와 다른 유형의 값을 사용 하여 본문을 선언할 수 있습니다 .

유효한 필드/속성 이름이 무엇인지 미리 알 필요 없이(Pydantic 모델의 경우처럼).

이것은 아직 모르는 키를 수신하려는 경우에 유용합니다.

다른 유용한 경우는 다른 유형의 키(예: int.

그것이 우리가 여기서 볼 것입니다.

이 경우 값 dict이 있는 int키가 있는 한 모두 허용합니다.float
```
from typing import Dict
from fastapi import FastAPI

app = FastAPI()

@app.post("/index-weights/")
async def create_index_weights(weights: Dict[int, float]):
    return weights
```
> strJSON 은 키로 만 지원 합니다.<br>그러나 Pydantic에는 자동 데이터 변환이 있습니다.<br>즉, API 클라이언트가 문자열을 키로만 보낼 수 있지만 해당 문자열에 순수한 정수가 포함되어 있으면 Pydantic이 변환하고 유효성을 검사합니다.<br>그리고 dict당신 weights은 실제로 int키와 float값을 가질 것입니다.

### 요약

FastAPI 를 사용하면 코드를 단순하고 짧고 우아하게 유지하면서 Pydantic 모델이 제공하는 최대의 유연성을 얻을 수 있습니다.

그러나 모든 이점이 있습니다.

- 에디터 지원(어디서나 완성!)
- 데이터 변환(파싱/직렬화라고도 함)
- 데이터 유효성 검사
- 스키마 문서
- 자동 문서