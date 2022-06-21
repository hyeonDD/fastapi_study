# 경로 매개변수

## 순서문제

- 상황
    - 경로 동작을 만들때 고정 경로를 갖고 있는 상황들을 맞닦뜨릴 수 있습니다.
    - /users/me처럼, 현재 사용자의 데이터를 가져온다고 합시다.
    - 사용자 ID를 이용해 특정 사용자의 정보를 가져오는 경로 /users/{user_id}도 있습니다.

- 해결방법
    - /users/me 가 /users/{user_id} 보다 먼저 오게 작성해야함.

코드
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}

@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```

## 사전 정의값

- 상황
    - 만약 경로 매개변수를 받는 경로 동작이 있지만, 유효하고 미리 정의할 수 있는 경로 매개변수 값을 원한다면 파이썬 표준 Enum을 사용할 수 있습니다.

- 해결방법
    - Enum 클래스 생성 (열거형클래스 파이썬 3.4이상 지원)
    - Enum을 임포트하고 str과 Enum을 상속하는 서브 클래스를 만듭니다.
    - str을 상속함으로써 API 문서는 값이 string 형이어야 하는 것을 알게 되고 제대로 렌더링 할 수 있게 됩니다.
    - 고정값으로 사용할 수 있는 유효한 클래스 어트리뷰트를 만들기.

코드
```
from enum import Enum
from fastapi import FastAPI

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name == ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```
> enum 클래스 값가져오는 방법은 model_name == ModelName.alexnet or model_name.value == "lenet" 이 있다.


## 경로를 포함하는 경로 매개변수¶

- 상황
    - /files/{file_path}가 있는 경로 동작이 있다고 해봅시다.
    - 그런데 여러분은 home/johndoe/myfile.txt처럼 path에 들어있는 file_path 자체가 필요합니다.
    - 따라서 해당 파일의 URL은 다음처럼 됩니다: /files/home/johndoe/myfile.txt.

- 해결방안
    - 그럼에도 Starlette의 내부 도구중 하나를 사용하여 FastAPI에서는 할 수 있습니다.
    - 매개변수에 경로가 포함되어야 한다는 문서를 추가하지 않아도 문서는 계속 작동합니다.
    - Starlette에서 직접 옵션을 사용하면 다음과 같은 URL을 사용하여 path를 포함하는 경로 매개변수를 선언 할 수 있습니다:
    - /files/{file_path:path}
    - 이러한 경우 매개변수의 이름은 file_path이고 마지막 부분 :path는 매개변수가 경로와 일치해야함을 알려줍니다.

코드
```
from fastapi import FastAPI

app = FastAPI()

@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```
> 매개변수가 /home/johndoe/myfile.txt를 갖고 있어 슬래시로 시작(/)해야 할 수 있습니다.
> 이 경우 URL은: /files//home/johndoe/myfile.txt이며 files과 home 사이에 이중 슬래시(//)가 생깁니다.

### 요약

FastAPI과 함께라면 짧고 직관적인 표준 파이썬 타입 선언을 사용하여 다음을 얻을 수 있습니다:

- 편집기 지원: 오류 검사, 자동완성 등
- 데이터 "파싱"
- 데이터 검증
- API 주석(Annotation)과 자동 문서

위 사항들을 그저 한번에 선언하면 됩니다.

이는 (원래 성능과는 별개로) 대체 프레임워크와 비교했을 때 FastAPI의 주요 가시적 장점일 것입니다.