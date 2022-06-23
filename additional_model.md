# 추가 모델

이전 예를 계속하면 하나 이상의 관련 모델이 있는 것이 일반적입니다.

다음과 같은 이유로 사용자 모델의 경우 특히 그렇습니다.

- 입력 모델 은 암호를 가질 수 있어야 합니다.
- 출력 모델 에는 암호가 없어야 합니다.
- 데이터베이스 모델 에는 해시된 암호가 있어야 합니다.

## 여러 모델

다음은 모델이 비밀번호 필드와 사용되는 장소와 함께 어떻게 생겼는지에 대한 일반적인 아이디어입니다.

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

class UserInDB(BaseModel):
    username: str
    hashed_password: str
    email: EmailStr
    full_name: Union[str, None] = None

def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password

def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db

@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

## 중복 감소

코드 중복을 줄이는 것은 FastAPI 의 핵심 아이디어 중 하나입니다 .

코드 중복으로 인해 버그, 보안 문제, 코드 비동기화 문제(한 곳에서는 업데이트하지만 다른 곳에서는 업데이트하지 않는 경우) 등의 가능성이 증가합니다.

그리고 이러한 모델은 모두 많은 데이터를 공유하고 속성 이름과 유형을 복제합니다.

우리는 더 잘할 수 있습니다.

UserBase다른 모델의 기반이 되는 모델을 선언할 수 있습니다. 그런 다음 해당 속성(유형 선언, 유효성 검사 등)을 상속하는 해당 모델의 하위 클래스를 만들 수 있습니다.

모든 데이터 변환, 유효성 검사, 문서화 등은 여전히 ​​정상적으로 작동합니다.

그렇게 하면 모델 간의 차이점만 선언할 수 있습니다(일반 텍스트 포함 password, hashed_password암호 포함 및 제외).

```
from typing import Union
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()

class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: Union[str, None] = None

class UserIn(UserBase):
    password: str

class UserOut(UserBase):
    pass

class UserInDB(UserBase):
    hashed_password: str

def fake_password_hasher(raw_password: str):
    return "supersecret" + raw_password

def fake_save_user(user_in: UserIn):
    hashed_password = fake_password_hasher(user_in.password)
    user_in_db = UserInDB(**user_in.dict(), hashed_password=hashed_password)
    print("User saved! ..not really")
    return user_in_db

@app.post("/user/", response_model=UserOut)
async def create_user(user_in: UserIn):
    user_saved = fake_save_user(user_in)
    return user_saved
```

## Union또는anyOf

응답을 두 가지 유형으로 선언할 수 있습니다. Union즉, 응답은 둘 중 하나가 됩니다.

로 OpenAPI에 정의됩니다 anyOf.

그렇게 하려면 표준 Python 유형 힌트를 사용하세요 

> 를 정의할 때 Union가장 구체적인 유형을 먼저 포함하고 덜 구체적인 유형을 포함합니다. 아래 예에서 보다 구체적인 것은 에서 PlaneItem앞에 나옵니다 .CarItemUnion[PlaneItem, CarItem]

```
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class BaseItem(BaseModel):
    description: str
    type: str

class CarItem(BaseItem):
    type = "car"

class PlaneItem(BaseItem):
    type = "plane"
    size: int

items = {
    "item1": {"description": "All my friends drive a low rider", "type": "car"},
    "item2": {
        "description": "Music is my aeroplane, it's my aeroplane",
        "type": "plane",
        "size": 5,
    },
}

@app.get("/items/{item_id}", response_model=Union[PlaneItem, CarItem])
async def read_item(item_id: str):
    return items[item_id]
```

## 모델 목록

같은 방법으로 개체 목록의 응답을 선언할 수 있습니다.

이를 위해 표준 Python typing.List(또는 listPython 3.9 이상에서만)을 사용하십시오.

```
from typing import List
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    name: str
    description: str

items = [
    {"name": "Foo", "description": "There comes my hero"},
    {"name": "Red", "description": "It's my aeroplane"},
]

@app.get("/items/", response_model=List[Item])
async def read_items():
    return items
```

## 임의의 응답dict

dictPydantic 모델을 사용하지 않고 키와 값의 유형만 선언 하는 일반 임의를 사용하여 응답을 선언할 수도 있습니다 .

이것은 유효한 필드/속성 이름(Pydantic 모델에 필요함)을 미리 모르는 경우에 유용합니다.

이 경우 다음을 사용할 수 있습니다 typing.Dict(또는 dictPython 3.9 이상에서만).

```
from typing import Dict
from fastapi import FastAPI

app = FastAPI()

@app.get("/keyword-weights/", response_model=Dict[str, float])
async def read_keyword_weights():
    return {"foo": 2.3, "bar": 3.4}
```

### 요약

여러 Pydantic 모델을 사용하고 각 사례에 대해 자유롭게 상속합니다.

엔터티가 다른 "상태"를 가질 수 있어야 하는 경우 엔터티당 단일 데이터 모델이 필요하지 않습니다. password를 포함 하고 password_hash암호가 없는 상태의 사용자 "엔티티"의 경우와 같습니다.