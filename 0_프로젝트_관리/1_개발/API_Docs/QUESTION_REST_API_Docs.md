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
|GET|https://mumulbo.com/api/v1/questions|질문 목록 조회|
|GET|https://mumulbo.com/api/v1/questions/{questionId}|질문 단건 조회|
|POST|https://mumulbo.com/api/v1/questions|질문 생성|
|PUT|https://mumulbo.com/api/v1/questions|질문 수정|
|DELETE|https://mumulbo.com/api/v1/questions/{questionId}|질문 삭제|

<br/>

---

### 2.1. 질문 목록 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/questions|-|

### 2.1.1. Request

#### 2.1.1.1. Header
N/A

#### 2.1.1.2. Path Variables
N/A

#### 2.1.1.3. Params
|Key|Type|Required|Default|Description|
|---|----|--------|-------|-----------|
|`lastId`|Long|❌|N/A|직전 페이지의 마지막 질문의 고유 번호(첫 페이지 요청 시 생략)|
|`size`|int|❌|`10`|요청할 질문 개수(최소 1개, 최대 1000개)|
|`authorId`|Long|❌|N/A|작성자로 필터 시 작성자 고유 번호|

#### 2.1.1.4. Body
N/A


### 2.1.2. Response

#### 2.1.2.1. Header
N/A

#### 2.1.2.2. Body

|Key|Type|Description|
|---|----|-----------|
|`questions`|list|현재 페이지 질문 목록(질문 id  내림차순)|
|`hasNext`|boolean|다음 페이지 존재 여부|
|`lastId`|Long|현재 페이지 마지막 질문의 고유 번호(다음 페이지 요청 시 사용)|
|`totalElements`|Long|질문 전체 개수|
|`pageSize`|int|실제 응답한 질문 개수|

### 2.1.3. Syntax
#### 2.1.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/questions?lastId=13&size=2&authorId=123
```

#### 2.1.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "questions": [
        {
            "id": 12,
            "subject": "질문입니다.",
            "content": "내용입니다.",
            "author": {
                "id": 123,
                "nickname": "heejaykong"
            },
            "createdAt": "2025-05-07T23:09:21.899321",
            "editedAt": "2025-05-07T23:09:21.899321"
        },
        {
            "id": 11,
            "subject": "질문입니다.",
            "content": "내용입니다.",
            "author": {
                "id": 123,
                "nickname": "heejaykong"
            },
            "createdAt": "2025-05-07T23:09:21.179749",
            "editedAt": "2025-05-07T23:09:21.179749"
        }
    ],
    "hasNext": true,
    "lastId": 11,
    "totalElements": 18,
    "pageSize": 2
}
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
|author|<pre lang="json">{&#13;  "id": 123,&#13;  "nickname": "heejaykong"&#13;}</pre>|작성자 정보|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|editedAt|"2025-05-06T23:26:04.436588"|수정 시간|

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
    "author": {
        "id": 321,
        "nickname": "heejaykong"
    },
    "createdAt": "2025-05-06T23:26:04.436588",
    "editedAt": "2025-05-06T23:26:04.436588"
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

|Key|Value|Required|Description|
|------|---|---|---|
|X-User-Id|123|O|작성자 고유 번호|

#### 2.3.1.2. Path Variables
N/A

#### 2.3.1.3. Params
N/A

#### 2.3.1.4. Body

|Key|Value|Required|Description|
|------|---|---|---|
|subject|"제목입니다."|O|질문 제목|
|content|"내용입니다."|O|질문 내용|

### 2.3.2. Response

#### 2.3.2.1. Header
N/A

#### 2.3.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|3|질문 고유 번호|
|subject|"제목입니다."|질문 제목|
|content|"내용입니다."|질문 내용|
|author|<pre lang="json">{&#13;  "id": 123,&#13;  "nickname": "heejaykong"&#13;}</pre>|작성자 정보|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|editedAt|"2025-05-06T23:26:04.436588"|수정 시간|

### 2.3.3. Syntax

#### 2.3.3.1 Request Syntax

```json
POST /api/v1/questions
Content-Type: application/json
X-User-Id: 123

{
    "subject": "제목입니다.",
    "content": "내용입니다."
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
    "author": {
        "id": 123,
        "nickname": "heejaykong"
    },
    "createdAt": "2025-05-06T23:26:04.436588",
    "editedAt": "2025-05-06T23:26:04.436588"
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

### 2.4.2. Response

#### 2.4.2.1. Header
N/A

#### 2.4.2.2. Body

|Key|Value|Description|
|------|---|---|
|id|1|질문 고유 번호|
|subject|"수정된 제목입니다."|질문 제목|
|content|"수정된 내용입니다."|질문 내용|
|author|<pre lang="json">{&#13;  "id": 123,&#13;  "nickname": "heejaykong"&#13;}</pre>|작성자 정보|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|editedAt|"2025-05-06T23:33:04.333333"|수정 시간|

### 2.4.3. Syntax

#### 2.4.3.1 Request Syntax

```json
PUT https://mumulbo.com/api/v1/questions
Content-Type: application/json

{
    "id": 3,
    "subject": "수정된 제목입니다.",
    "content": "수정된 내용입니다.",
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
    "author": {
        "id": 321,
        "nickname": "heejaykong"
    },
    "createdAt": "2025-05-06T23:26:04.436588",
    "editedAt": "2025-05-06T23:33:04.333333"
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
