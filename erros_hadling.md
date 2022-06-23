# 오류 처리

API를 사용하는 클라이언트에 오류를 알려야 하는 상황이 많이 있습니다.

이 클라이언트는 프론트엔드가 있는 브라우저, 다른 사람의 코드, IoT 장치 등이 될 수 있습니다.

클라이언트에게 다음과 같이 알려야 할 수 있습니다.

- 클라이언트에 해당 작업에 대한 충분한 권한이 없습니다.
- 클라이언트는 해당 리소스에 액세스할 수 없습니다.
- 클라이언트가 액세스하려는 항목이 존재하지 않습니다.
- 등.
이러한 경우 일반적으로 400 (400~499) 범위의 HTTP 상태 코드 를 반환합니다.

이것은 200 HTTP 상태 코드(200에서 299까지)와 유사합니다. 이 "200" 상태 코드는 어떻게든 요청에 ​​"성공"이 있었음을 의미합니다.

400 범위의 상태 코드는 클라이언트에서 오류가 있음을 의미합니다.

"404 Not Found" 오류(및 농담)를 모두 기억 하십니까?

## HTTPException 사용법

사용하는 클라이언트에 오류가 있는 HTTP 응답을 반환하려면 HTTPException.

## import HTTPException

```
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

## HTTPException코드에서 올리기

HTTPExceptionAPI와 관련된 추가 데이터가 있는 일반적인 Python 예외입니다.

그것은 파이썬 예외이기 때문에, 당신은 return그것을 하지 않습니다 raise.

이것은 또한 경로 연산 함수 내부에서 호출하는 유틸리티 함수 내부에 있고 해당 유틸리티 함수 내부에서 발생 시키는 경우 경로 연산 함수HTTPException 의 나머지 코드를 실행하지 않는다는 것을 의미합니다. 해당 요청을 즉시 종료하고 HTTP 오류를 클라이언트에서 보냅니다.HTTPException

값보다 예외를 발생시키는 이점은 return종속성과 보안에 대한 섹션에서 더 분명해질 것입니다.

이 예에서 클라이언트가 존재하지 않는 ID로 항목을 요청하면 다음과 같은 상태 코드로 예외를 발생시킵니다 404.

```
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

## 사용자 정의 헤더 추가

HTTP 오류에 사용자 정의 헤더를 추가할 수 있는 것이 유용한 몇 가지 상황이 있습니다. 예를 들어, 일부 보안 유형의 경우.

코드에서 직접 사용할 필요는 없을 것입니다.

그러나 고급 시나리오에 필요한 경우 사용자 정의 헤더를 추가할 수 있습니다.

```
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}

@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

## 사용자 정의 예외 처리기 설치

Starlette의 동일한 예외 유틸리티를 사용하여 사용자 정의 예외 처리기를 추가할 수 있습니다.

UnicornException당신 (또는 당신이 사용하는 라이브러리)이 할 수 있는 사용자 정의 예외가 있다고 가정 해 봅시다 raise.

그리고 FastAPI를 사용하여 이 예외를 전역적으로 처리하려고 합니다.

다음 을 사용하여 사용자 정의 예외 처리기를 추가할 수 있습니다 @app.exception_handler().

```
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

app = FastAPI()

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

여기에서 요청 하면 /unicorns/yolo경로 작업 은 raise.UnicornException

그러나 그것은 에 의해 처리됩니다 unicorn_exception_handler.

따라서 다음과 같은 HTTP 상태 코드 418와 JSON 콘텐츠가 포함된 깨끗한 오류가 수신됩니다.

> from starlette.requests import Request및 를 사용할 수도 있습니다 from starlette.responses import JSONResponse.<br>FastAPI 는 개발자에게 편의 와 starlette.responses같은 것을 제공합니다. fastapi.responses그러나 사용 가능한 대부분의 응답은 Starlette에서 직접 제공됩니다. 와 동일합니다 Request.

## 기본 예외 처리기 재정의

FastAPI 에는 몇 가지 기본 예외 처리기가 있습니다.

이러한 핸들러는 요청에 잘못된 데이터가 있을 raise때 기본 JSON 응답을 반환하는 역할을 합니다.HTTPException

이러한 예외 처리기를 자신의 것으로 재정의할 수 있습니다.

## 요청 유효성 검사 예외 재정의

요청에 잘못된 데이터가 포함된 경우 FastAPI 는 내부적으로 RequestValidationError.

또한 이에 대한 기본 예외 처리기가 포함되어 있습니다.

재정의하려면 가져오고 RequestValidationError와 함께 사용 @app.exception_handler(RequestValidationError)하여 예외 처리기를 장식합니다.

예외 처리기는 Request및 예외를 수신합니다.

```
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

## HTTPException오류 처리기 재정의

HTTPException같은 방법으로 핸들러 를 재정의할 수 있습니다 .

예를 들어 다음 오류에 대해 JSON 대신 일반 텍스트 응답을 반환할 수 있습니다.

```
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

## RequestValidationError몸 을 사용

유효하지 않은 데이터와 함께 수신 된 내용 이 RequestValidationError포함되어 있습니다 .body

앱을 개발하는 동안 본문을 기록하고 디버그하고 사용자에게 반환하는 등의 작업을 수행하는 동안 사용할 수 있습니다.

```
from fastapi import FastAPI, Request, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({"detail": exc.errors(), "body": exc.body}),
    )

class Item(BaseModel):
    title: str
    size: int

@app.post("/items/")
async def create_item(item: Item):
    return item
```

## FastAPI 의 예외 처리기 재사용

FastAPI 의 동일한 기본 예외 핸들러와 함께 예외를 사용하려면 다음에서 기본 예외 핸들러를 가져와서 재사용할 수 있습니다 fastapi.exception_handlers.

```
from fastapi import FastAPI, HTTPException
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    return await http_exception_handler(request, exc)

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    print(f"OMG! The client sent invalid data!: {exc}")
    return await request_validation_exception_handler(request, exc)

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

이 예에서는 print매우 표현적인 메시지로 오류를 표시하고 있지만 아이디어는 알 수 있습니다. 예외를 사용한 다음 기본 예외 처리기를 다시 사용할 수 있습니다.