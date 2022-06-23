# 폼 데이터

## import Form

Form다음 에서 가져오기 fastapi:

```
from fastapi import FastAPI, Form

app = FastAPI()

@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

## Form매개변수 정의

Body또는 와 동일한 방식으로 양식 매개변수를 작성하십시오 Query.

```
from fastapi import FastAPI, Form
app = FastAPI()

@app.post("/login/")
async def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

username예를 들어, OAuth2 사양을 사용할 수 있는 방법 중 하나로("암호 흐름"이라고 함) 및 password양식 필드 를 보내야 합니다.

사양 에 따르면 필드 의 이름은 정확하게 지정해야 하며 usernameJSON password이 아닌 양식 필드로 보내야 합니다.

를 사용 하면 (및 , , ) Form와 동일한 메타데이터 및 유효성 검사를 선언할 수 있습니다 .BodyQueryPathCookie

> Form에서 직접 상속하는 클래스입니다 Body.
> Form양식 본문을 선언하려면 매개변수가 쿼리 매개변수 또는 본문(JSON) 매개변수로 해석되기 때문에 명시적으로 사용해야 합니다 .

## "양식 필드" 정보

HTML 양식( <form></form>)이 서버에 데이터를 보내는 방식은 일반적으로 해당 데이터에 대해 "특수" 인코딩을 사용하며 JSON과 다릅니다.

FastAPI 는 JSON 대신 올바른 위치에서 해당 데이터를 읽도록 합니다.

> 양식의 데이터는 일반적으로 "미디어 유형"을 사용하여 인코딩됩니다 application/x-www-form-urlencoded.<br>그러나 양식에 파일이 포함된 경우 로 인코딩됩니다 multipart/form-data. 다음 장에서 파일 처리에 대해 읽을 것입니다.<br>이러한 인코딩 및 양식 필드에 대해 자세히 알아보려면 에 대한 MDN 웹 문서로POST 이동하십시오 .

> 경로 작업 에서 여러 Form매개변수를 선언할 수 있지만 요청 에 대신 을 사용하여 인코딩된 본문이 있으므로 JSON으로 수신할 것으로 예상되는 필드 도 선언할 수 없습니다 .Bodyapplication/x-www-form-urlencodedapplication/json<br>이것은 FastAPI 의 제한 사항이 아니라 HTTP 프로토콜의 일부입니다.

### 요약

Form양식 데이터 입력 매개변수를 선언하는 데 사용 합니다.