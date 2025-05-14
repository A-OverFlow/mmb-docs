# 무물보 API 문서 - Answer Service

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

|HTTP Status Code|Custom Error Code|Description|응답 예시(Body)|
|------|---|---|---|
|400 BAD_REQUEST|`ANSWER-4000`|잘못된 요청|<pre lang="json">{&#13;  "code": "",&#13;  "message": &#13;}</pre>|
|404 NOT_FOUND|`ANSWER-4004`|답변 Not Found|<pre lang="json">{&#13;  "code": "",&#13;  "message": &#13;}</pre>|
|500 INTERNAL_SERVER_ERROR|`ANSWER-5000`|답변 서버 에러|<pre lang="json">{&#13;  "code": "",&#13;  "message": &#13;}</pre>|

<br/><br/>

---

## 2. Answer Service

### 2.0. API 리스트

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions/{questionId}/answers|질문의 답변 전체 조회|
|GET|https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}|답변 단건 조회|
|POST|https://mumulbo.com/api/v1/questions/{questionId}/answers|답변 생성|
|PUT|https://mumulbo.com/api/v1/questions/{questionId}/answers|답변 수정|
|DELETE|https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}|답변 삭제|

<br/>

---

### 2.1. 질문의 답변 전체 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions/{questionId}/answers|-|

### 2.1.1. Request

#### 2.1.1.1. Header
N/A

#### 2.1.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|

#### 2.1.1.3. Params
N/A

#### 2.1.1.4. Body
N/A

### 2.1.2. Response

#### 2.1.2.1. Header
N/A

#### 2.1.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|답변 고유 번호|
|content|"답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|comments|[]|댓글 목록|
|emotions|[]|이모지 목록|

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/questions/{questionId}/answers
```

#### 2.1.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

[
    {
        "id": 1,
        "content": "답변입니다.",
        "author": "joonsub.lim",
        "status": "ACCEPTED",
        "comments": [],
        "emotions": []
    },
    {
        "id": 2,
        "content": "또 다른 답변입니다.",
        "author": "heejaykong",
        "status": "NOT_ACCEPTED",
        "comments": [],
        "emotions": []
    }
]
```

<br/>

---

### 2.2. 답변 단건 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}|-|

### 2.2.1. Request

#### 2.2.1.1. Header
N/A

#### 2.2.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|
|answerId|답변 고유 번호|

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
|id|1|답변 고유 번호|
|content|"답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|comments|[]|댓글 목록|
|emotions|[]|이모지 목록|

### 2.2.3. Syntax

#### 2.2.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}
```

#### 2.2.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 1,
    "content": "답변입니다.",
    "author": "joonsub.lim",
    "status": "ACCEPTED",
    "comments": [],
    "emotions": []
}
```

<br/>

---

### 2.3. 답변 생성

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/questions/{questionId}/answers|-|

### 2.3.1. Request

#### 2.3.1.1. Header
N/A

#### 2.3.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|

#### 2.3.1.3. Params
N/A

#### 2.3.1.4. Body

|Key|Value|Description|
|------|---|---|
|content|"답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|

### 2.3.2. Response

#### 2.3.2.1. Header
N/A

#### 2.3.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|답변 고유 번호|
|content|"답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|comments|[]|댓글 목록|
|emotions|[]|이모지 목록|

### 2.3.3. Syntax

#### 2.3.3.1 Request Syntax

```json
POST https://mumulbo.com/api/v1/questions/{questionId}/answers

{
    "content": "답변입니다.",
    "author": "joonsub.lim",
}
```

#### 2.3.3.2. Response Syntax

```json
HTTP/1.1 201 Created
Content-Type: application/json
Location: /questions/{questionId}/answers/{answerId}

{
    "id": 1,
    "content": "답변입니다.",
    "author": "joonsub.lim",
    "status": "ACCEPTED",
    "comments": [],
    "emotions": []
}
```

<br/>

---

### 2.4. 답변 수정

|HTTP Method|URL|비고|
|------|---|---|
|PUT|https://mumulbo.com/api/v1/questions/{questionId}/answers|-|

### 2.4.1. Request

#### 2.4.1.1. Header
N/A

#### 2.4.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|

#### 2.4.1.3. Params
N/A

#### 2.4.1.4. Body

|Key|Value|Description|
|------|---|---|
|id|1|답변 고유 번호|
|content|"수정된 답변입니다."|답변 내용|
|status|"NOT_ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|

### 2.4.2. Response

#### 2.4.2.1. Header
N/A

#### 2.4.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|답변 고유 번호|
|content|"수정된 답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"NOT_ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|comments|[]|댓글 목록|
|emotions|[]|이모지 목록|

### 2.4.3. Syntax

#### 2.4.3.1 Request Syntax

```json
PUT https://mumulbo.com/api/v1/questions/{questionId}/answers
Content-Type: application/json

{
    "id": 1,
    "content": "수정된 답변입니다.",
    "status": "NOT_ACCEPTED",
}
```

#### 2.4.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 1,
    "content": "수정된 답변입니다.",
    "author": "joonsub.lim",
    "status": "NOT_ACCEPTED",
    "comments": [],
    "emotions": []
}
```

<br/>

---

### 2.5. 답변 삭제

|HTTP Method|URL|비고|
|------|---|---|
|DELETE|https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}|-|

### 2.5.1. Request

#### 2.5.1.1. Header
N/A

#### 2.5.1.2. Path Variables

|Key|Description|
|------|---|
|questionId|질문 고유 번호|
|answerId|답변 고유 번호|

#### 2.5.1.3. Params
N/A

#### 2.5.1.4. Body
N/A

### 2.5.2. Response
N/A

### 2.5.3. Syntax

#### 2.5.3.1 Request Syntax

```json
DELETE https://mumulbo.com/api/v1/questions/{questionId}/answers/{answerId}
```

#### 2.5.3.2. Response Syntax

```json
HTTP/1.1 204 No Content
```

<br/>

---
