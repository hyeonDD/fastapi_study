# Cookie Parameters

## import cookie

```
from typing import Union

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Union[str, None] = Cookie(default=None)):
    return {"ads_id": ads_id}
```

## Cookie매개변수 선언

Path그런 다음 및 와 동일한 구조를 사용하여 쿠키 매개변수를 선언합니다 Query.

첫 번째 값은 기본값이며 모든 추가 유효성 검사 또는 주석 매개변수를 전달할 수 있습니다.

```
from typing import Union
from fastapi import Cookie, FastAPI

app = FastAPI()

@app.get("/items/")
async def read_items(ads_id: Union[str, None] = Cookie(default=None)):
    return {"ads_id": ads_id}
```

> CookiePath및 의 "자매" 클래스입니다 Query. 또한 동일한 공통 Param클래스에서 상속됩니다.<br>Query그러나 에서 Path, Cookie및 기타 를 가져올 때 fastapi실제로는 특수 클래스를 반환하는 함수라는 것을 기억하십시오.
> Cookie쿠키를 선언하려면 매개변수가 쿼리 매개변수로 해석되기 때문에 를 사용해야 합니다 .

### 요약
QueryPath 및 와 Cookie동일한 공통 패턴을 사용하여 로 쿠키를 선언합니다.