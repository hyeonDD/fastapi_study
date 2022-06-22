# 쿼리 매개변수 및 문자열 유효성 검사

예시 코드
```
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[str, None] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

쿼리 매개변수 q는 유형 Union[str, None](또는 str | NonePython 3.10에서)입니다. 즉, 유형 str이지만 가 될 수도 있으며 None실제로 기본값은 None이므로 FastAPI는 필수가 아님을 알 수 있습니다.

## 추가 검증

q선택적이지만 제공될 때마다 길이가 50자를 초과하지 않도록 강제할 것 입니다.

import Query
이를 달성하려면 먼저 다음에서 가져 Query옵니다 fastapi.

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```
> Query(default=None, max_length=50) 쿼리의 최대 길이는 50까지 가능

## Query기본값으로 사용

이제 이를 매개변수의 기본값으로 사용하고 매개변수 max_length를 50으로 설정합니다.

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=None, max_length=50)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

None함수 의 기본값을 로 바꿔야 하므로 Query()이제 매개변수로 기본값을 설정할 수 있습니다 Query(default=None). 기본값을 정의하는 것과 동일한 목적을 수행합니다.

그래서:

q: Union[str, None] = Query(default=None)
...다음과 같이 매개변수를 선택 사항으로 만듭니다.

q: Union[str, None] = None
그리고 Python 3.10 이상에서:

q: str | None = Query(default=None)
...다음과 같이 매개변수를 선택 사항으로 만듭니다.

q: str | None = None
그러나 쿼리 매개변수로 명시적으로 선언합니다.

> 매개변수를 선택 사항으로 만드는 가장 중요한 부분은 다음과 같은 부분입니다.<br>= None<br>아니면 그:<br>= Query(default=None)<br>None기본값으로 사용하므로 매개변수 가 필요하지 않습니다 . 이 Union[str, None]부분을 사용하면 편집기에서 더 나은 지원을 제공할 수 있지만 FastAPI에 이 매개변수가 필요하지 않다고 알려주는 것은 아닙니다.

그런 다음 더 많은 매개변수를 에 전달할 수 있습니다 Query. 이 경우 max_length문자열에 적용되는 매개변수는 다음과 같습니다.

```
q: Union[str, None] = Query(default=None, max_length=50)
```
이렇게 하면 데이터의 유효성을 검사하고 데이터가 유효하지 않은 경우 명확한 오류를 표시하며 OpenAPI 스키마 경로 작업 의 매개변수를 문서화합니다.

## 더 많은 유효성 검사 추가
매개변수를 추가할 수도 있습니다 min_length.

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(default=None, min_length=3, max_length=50)
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 정규식 추가

매개변수가 일치해야 하는 정규식 을 정의할 수 있습니다 .

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None, min_length=3, max_length=50, regex="^fixedquery$"
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

이 특정 정규식은 수신된 매개변수 값이 다음과 같은지 확인합니다.

- ^: 다음 문자로 시작하며 이전에는 문자가 없습니다.
- fixedquery: 정확한 값이 fixedquery있습니다.
- $: 여기서 끝납니다. 이후에는 더 이상 문자가 없습니다 fixedquery.
이 모든 "정규 표현식" 아이디어에 대해 길을 잃는다고 느끼더라도 걱정하지 마십시오. 그들은 많은 사람들에게 어려운 주제입니다. 아직 정규 표현식 없이도 많은 일을 할 수 있습니다.

그러나 필요할 때마다 가서 배우면 이미 FastAPI 에서 직접 사용할 수 있습니다 .

## 기본값
None매개변수 의 값으로 전달할 수 있는 것과 같은 방식으로 default다른 값을 전달할 수 있습니다.

q쿼리 매개변수를 선언하고 다음 과 같은 기본값을 갖고 min_length싶다고 가정해 보겠습니다.
3"fixedquery"

```
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(default="fixedquery", min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```
>기본값이 있으면 매개변수도 선택사항이 됩니다.

## 필수로 만드세요

```
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

### 줄임표( ...) 와 함께 필수

값이 필요함을 명시적으로 선언하는 다른 방법이 있습니다. default매개변수를 리터럴 값으로 설정할 수 있습니다 (...)

```
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(default=..., min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

> "Ellipsis"라고 합니다.[...docs](https://docs.python.org/3/library/constants.html#Ellipsis)

## 필수None
None매개변수가 허용할 수 있지만 여전히 필수라고 선언할 수 있습니다 . 이렇게 하면 값이 인 경우에도 클라이언트가 값을 보내도록 합니다 None.

그렇게 하려면 유효한 유형이라고 선언할 수 None있지만 여전히 다음을 사용합니다 default=....

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=..., min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## Required줄임표( ...) 대신 Pydantic 을 사용하십시오.
사용이 불편하다면 Pydantic에서 ...가져와서 사용할 수도 있습니다 .Required

```
from fastapi import FastAPI, Query
from pydantic import Required

app = FastAPI()

@app.get("/items/")
async def read_items(q: str = Query(default=Required, min_length=3)):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 쿼리 매개변수 목록/여러 값

쿼리 매개변수를 명시적으로 정의하면 Query값 목록을 받도록 선언하거나 다른 방식으로 여러 값을 받을 수도 있습니다.
예를 들어 qURL에 여러 번 나타날 수 있는 쿼리 매개변수를 선언하려면 다음과 같이 작성할 수 있습니다.

```
from typing import List, Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[List[str], None] = Query(default=None)):
    query_items = {"q": q}
    return query_items
```

그런 다음 다음과 같은 URL을 사용합니다.

```
http://localhost:8000/items/?q=foo&q=bar
```

## 쿼리 매개변수 목록/기본값이 있는 여러 값
list제공되지 않은 경우 기본값을 정의할 수도 있습니다 .

```
from typing import List
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: List[str] = Query(default=["foo", "bar"])):
    query_items = {"q": q}
    return query_items
```

## 더 많은 메타데이터 선언
매개변수에 대한 추가 정보를 추가할 수 있습니다.

해당 정보는 생성된 OpenAPI에 포함되고 문서 사용자 인터페이스 및 외부 도구에서 사용됩니다.

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(default=None, title="Query string", min_length=3)
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

그리고 description:

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Union[str, None] = Query(
        default=None,
        title="Query string",
        description="Query string for the items to search in the database that have a good match",
        min_length=3,
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 별칭 매개변수
매개변수가 가 되기를 원한다고 상상해 보십시오 item-query.

에서처럼:

http://127.0.0.1:8000/items/?item-query=foobaritems
그러나 item-query유효한 Python 변수 이름이 아닙니다.

```
from typing import Union
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q: Union[str, None] = Query(default=None, alias="item-query")):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

## 요약
매개변수에 대한 추가 유효성 검사 및 메타데이터를 선언할 수 있습니다.

일반 유효성 검사 및 메타데이터:

- alias
- title
- description
- deprecated

문자열에 대한 유효성 검사:

- min_length
- max_length
- regex

이 예에서 str값에 대한 유효성 검사를 선언하는 방법을 보았습니다.

숫자와 같은 다른 유형에 대한 유효성 검사를 선언하는 방법을 보려면 다음 장을 참조하십시오.