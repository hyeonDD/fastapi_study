# 폼 및 파일 요청

File 과 Form 을 사용하여 파일과 폼을 함께 정의할 수 있습니다.

## File 및 Form 업로드

```
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
    file: bytes = File(), fileb: UploadFile = File(), token: str = Form()
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

## File 및 Form 매개변수 정의

Body 및 Query와 동일한 방식으로 파일과 폼의 매개변수를 생성합니다:

```
from fastapi import FastAPI, File, Form, UploadFile

app = FastAPI()

@app.post("/files/")
async def create_file(
    file: bytes = File(), fileb: UploadFile = File(), token: str = Form()
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```
파일과 폼 필드는 폼 데이터 형식으로 업로드되어 파일과 폼 필드로 전달됩니다.

어떤 파일들은 bytes로, 또 어떤 파일들은 UploadFile로 선언할 수 있습니다.

### 요약

하나의 요청으로 데이터와 파일들을 받아야 할 경우 File과 Form을 함께 사용하기 바랍니다.