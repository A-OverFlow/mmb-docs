# 무물보 API 문서 - Question Service

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

|HTTP Status Code|Custom Error Code|Description|응답 예시(Body)|
|------|---|---|---|
|**400** BAD_REQUEST|`QUESTION-4000`|잘못된 요청|<pre lang="json">{&#13;  "code": "QUESTION-4000",&#13;  "message": null&#13;}</pre>|
|**404** NOT_FOUND|`QUESTION-4040`|질문 not found|<pre lang="json">{&#13;  "code": "QUESTION-4040",&#13;  "message": null&#13;}</pre>|
|**500** INTERNAL_SERVER_ERROR|`QUESTION-5000`|질문 서버 에러|<pre lang="json">{&#13;  "code": "QUESTION-5000",&#13;  "message": null&#13;}</pre>|

<br/><br/>

---

## 2. Question Service

### 2.0. API 리스트

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions|질문 전체 조회|
|GET|https://mumulbo.com/api/v1/questions/{questionId}|질문 단건 조회|
|POST|https://mumulbo.com/api/v1/questions|질문 생성|
|PUT|https://mumulbo.com/api/v1/questions|질문 수정|
|DELETE|https://mumulbo.com/api/v1/questions/{questionId}|질문 삭제|

<br/>

---

### 2.1. 질문 전체 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions|-|

### 2.1.1. Request
N/A

### 2.1.2. Response

#### 2.1.2.1. Header
N/A

#### 2.1.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|질문 고유 번호|
|subject|"제목입니다."|질문 제목|
|content|"내용입니다."|질문 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"NEW"|질문 상태(NEW/ING/DONE)|
|answers|[]|답변 목록|

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/questions
```

#### 2.1.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

[
    {
        "id": 1,
        "subject": "제목입니다.",
        "content": "내용입니다.",
        "author": "joonsub.lim",
        "status": "NEW",
        "answers": [],
    },
    {
        "id": 2,
        "subject": "안녕하세요?",
        "content": "반갑습니다.",
        "author": "heejaykong",
        "status": "DONE",
        "answers": [],
    }
]
```

<br/>

---

### 2.2. 질문 단건 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions/{questionId}|-|

### 2.2.1. Request

#### 2.2.1.1. Header
N/A

#### 2.2.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|

#### 2.2.1.3. Params
N/A

#### 2.2.1.4. Body
N/A

### 2.2.2. Response

#### 2.2.2.1. Header
N/A

#### 2.2.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|질문 고유 번호|
|subject|"제목입니다."|질문 제목|
|content|"내용입니다."|질문 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"NEW"|질문 상태(NEW/ING/DONE)|
|answers|[]|답변 목록|

### 2.2.3. Syntax

#### 2.2.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/questions/{questionId}
```

#### 2.2.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 1,
    "subject": "제목입니다.",
    "content": "내용입니다.",
    "author": "joonsub.lim",
    "status": "NEW",
    "answers": []
}
```

<br/>

---

### 2.3. 질문 생성

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/questions|-|

### 2.3.1. Request

#### 2.3.1.1. Header
N/A

#### 2.3.1.2. Path Variables
N/A

#### 2.3.1.3. Params
N/A

#### 2.3.1.4. Body

|Key|Value|Required|Description|
|------|---|---|---|
|subject|"제목입니다."|O|질문 제목|
|content|"내용입니다."|O|질문 내용|
|author|"heejaykong"|O|작성자 이름|

### 2.3.2. Response

#### 2.3.2.1. Header
N/A

#### 2.3.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|3|질문 고유 번호|
|subject|"제목입니다."|질문 제목|
|content|"내용입니다."|질문 내용|
|author|"heejaykong"|작성자 이름|
|status|"NEW"|질문 상태(NEW/ING/DONE)|
|answers|[]|답변 목록|

### 2.3.3. Syntax

#### 2.3.3.1 Request Syntax

```json
POST https://mumulbo.com/api/v1/questions
Content-Type: application/json

{
    "subject": "제목입니다.",
    "content": "내용입니다.",
    "author": "heejaykong"
}
```

#### 2.3.3.2. Response Syntax

```json
HTTP/1.1 201 Created
Content-Type: application/json
Location: /questions/{id}

{
    "id": 3,
    "subject": "제목입니다.",
    "content": "내용입니다.",
    "author": "heejaykong",
    "status": "NEW",
    "answers": []
}
```

<br/>

---

### 2.4. 질문 수정

|HTTP Method|URL|비고|
|------|---|---|
|PUT|https://mumulbo.com/api/v1/questions|-|

### 2.4.1. Request

#### 2.4.1.1. Header
N/A

#### 2.4.1.2. Path Variables
N/A

#### 2.4.1.3. Params
N/A

#### 2.4.1.4. Body

|Key|Value|Required|Description|
|------|---|---|---|
|id|1|O|질문 고유 번호|
|subject|"수정된 제목입니다."|O|질문 제목|
|content|"수정된 내용입니다."|O|질문 내용|
|status|"ING"|O|질문 상태|

### 2.4.2. Response

#### 2.4.2.1. Header
N/A

#### 2.4.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|질문 고유 번호|
|subject|"수정된 제목입니다."|질문 제목|
|content|"수정된 내용입니다."|질문 내용|
|author|"heejaykong"|작성자 이름|
|status|"ING"|질문 상태(NEW/ING/DONE)|
|answers|[]|답변 목록|

### 2.4.3. Syntax

#### 2.4.3.1 Request Syntax

```json
PUT https://mumulbo.com/api/v1/questions
Content-Type: application/json

{
    "id": 3,
    "subject": "수정된 제목입니다.",
    "content": "수정된 내용입니다.",
    "status": "ING"
}
```

#### 2.4.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 3,
    "subject": "수정된 제목입니다.",
    "content": "수정된 내용입니다.",
    "author": "heejaykong",
    "status": "ING",
    "answers": []
}
```

<br/>

---

### 2.5. 질문 삭제

|HTTP Method|URL|비고|
|------|---|---|
|DELETE|https://mumulbo.com/api/v1/questions/{questionId}|-|

### 2.5.1. Request

#### 2.5.1.1. Header
N/A

#### 2.5.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|

#### 2.5.1.3. Params
N/A

#### 2.5.1.4. Body
N/A

### 2.5.2. Response
N/A

### 2.5.3. Syntax

#### 2.5.3.1 Request Syntax

```json
DELETE https://mumulbo.com/api/v1/questions/{questionId}
```

#### 2.5.3.2. Response Syntax

```json
HTTP/1.1 204 No Content
```

<br/>

---
