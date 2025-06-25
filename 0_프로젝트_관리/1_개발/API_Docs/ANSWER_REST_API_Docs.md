# 무물보 API 문서 - Answer Service

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

|HTTP Status Code|Custom Error Code|Description|응답 예시(Body)|
|------|---|---|---|
|400 BAD_REQUEST|`ANSWER-4000`|잘못된 요청|<pre lang="json">{&#13;  "code": "ANSWER-4000",&#13;  "message": null&#13;}</pre>|
|404 NOT_FOUND|`ANSWER-4004`|답변 Not Found|<pre lang="json">{&#13;  "code": "ANSWER-4004",&#13;  "message": null&#13;}</pre>|
|500 INTERNAL_SERVER_ERROR|`ANSWER-5000`|답변 서버 에러|<pre lang="json">{&#13;  "code": "ANSWER-5000",&#13;  "message": null&#13;}</pre>|

<br/><br/>

---

## 2. Answer Service

### 2.0. API 리스트

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/answers/{questionId}|질문의 답변 전체 조회|
|GET|~~https://mumulbo.com/api/v1/answers/{answerId}~~|~~답변 단건 조회~~ 스펙 아웃|
|POST|https://mumulbo.com/api/v1/answers|답변 생성|
|PATCH|https://mumulbo.com/api/v1/answers/{answerId}|답변 수정
|DELETE|https://mumulbo.com/api/v1/answers/{answerId}|답변 삭제
|PATCH|https://mumulbo.com/api/v1/answers/{answerId}/accept|채택(NEXT)
|GET|https://mumulbo.com/api/v1/recent|최근 답변
|GET|https://mumulbo.com/api/v1/count|답변 수
|GET|https://mumulbo.com/api/v1/|답변이 가장 많이 달린 질문(NEXT)
|GET|https://mumulbo.com/api/v1/|내가 작성한 답변의 질문 목록(NEXT)

<br/>

---

### 2.1. 질문의 답변 전체 조회

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/answers/{questionId}|-|

### 2.1.1. Request

#### 2.1.1.1. Header
Authorization Bearer {JWT}

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
|answerId|1|답변 고유 번호|
|content|"답변이다냥"|답변 내용|
|author|{"id": 123,"nickname": "yuyeonchoe","picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"}|작성자 정보|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|updatedAt|"2025-05-07T23:26:04.436588"|수정 시간|

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/answers/{questionId}
```

#### 2.1.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

[
    {
        "answerId": 1,
        "content": "답변이다냥.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "ACCEPTED",
        "createdAt": "2025-05-06T23:26:04.436588",
        "updatedAt": "2025-05-07T23:26:04.436588"
    },
    {
        "answerId": 2,
        "content": "다른 답변이다냥.",
        "author": {
                "id": 234,
                "nickname": "heejaykong",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77NH"
            },
        "status": "NOT_ACCEPTED"
        "createdAt": "2025-05-08T23:26:04.436588",
        "updatedAt": "2025-05-08T23:26:04.436588"
    }
]
```

<br/>

---

### 2.2. 답변 생성

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/answers|-|

### 2.2.1. Request

#### 2.2.1.1. Header
Authorization Bearer {JWT}

#### 2.2.1.2. Path Variables
N/A

#### 2.2.1.3. Params
N/A

#### 2.2.1.4. Body

|Key|Value|Description|
|------|---|---|
|questionId|1|질문 고유 번호|
|answer|"답변입니다."|답변 내용|

### 2.2.2. Response

#### 2.2.2.1. Header
N/A

#### 2.2.2.2. Body

|Key|Value|Description|
|------|---|---|
|answerId|1|답변 고유 번호|
|answer|"답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|updatedAt|"2025-05-06T23:26:04.436588"|수정 시간|

### 2.2.3. Syntax

#### 2.2.3.1 Request Syntax

```json
POST https://mumulbo.com/api/v1/answers

{
    "questionId" 1,
    "answer": "답변입니다."
}
```

#### 2.2.3.2. Response Syntax

```json
HTTP/1.1 201 Created
Content-Type: application/json
Location: https://mumulbo.com/api/v1/answers/1
{
    "answerId": 1,
    "answer": "답변입니다.",
    "author": "joonsub.lim",
    "status": "NOT_ACCEPTED"
    "createdAt": "2025-05-06T23:26:04.436588",
    "updatedAt": "2025-05-06T23:26:04.436588"
}
```

<br/>

---

### 2.3. 답변 수정

|HTTP Method|URL|비고|
|------|---|---|
|PATCH|https://mumulbo.com/api/v1/answers/{answerId}|-|

### 2.3.1. Request

#### 2.3.1.1. Header
Authorization Bearer {JWT}

#### 2.3.1.2. Path Variables

|Key|Description|
|------|---|
|answerId|답변 고유 번호|

#### 2.3.1.3. Params
N/A

#### 2.3.1.4. Body

|Key|Value|Description|
|------|---|---|
|answer|"수정된 답변입니다."|답변 내용|

### 2.3.2. Response

#### 2.3.2.1. Header
N/A

#### 2.3.2.2. Body

|Key|Value|Description|
|------|---|---|
|answerId|1|답변 고유 번호|
|answer|"수정된 답변입니다."|답변 내용|
|author|"joonsub.lim"|작성자 이름|
|status|"NOT_ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|updatedAt|"2025-05-07T23:26:04.436588"|수정 시간|

### 2.3.3. Syntax

#### 2.3.3.1 Request Syntax

```json
PUT https://mumulbo.com/api/v1/answers/{answerId}
Content-Type: application/json

{
    "answer": "수정된 답변입니다."
}
```

#### 2.3.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "answerId": 1,
    "answer": "수정된 답변입니다.",
    "author": "joonsub.lim",
    "status": "NOT_ACCEPTED",
    "createdAt": "2025-05-06T23:26:04.436588",
    "updatedAt": "2025-05-07T23:26:04.436588"
}
```

<br/>

---

### 2.4. 답변 삭제

|HTTP Method|URL|비고|
|------|---|---|
|DELETE|https://mumulbo.com/api/v1/answers/{answerId}|-|

### 2.4.1. Request

#### 2.4.1.1. Header
Authorization Bearer {JWT}

#### 2.4.1.2. Path Variables

|Key|Description|
|------|---|
|answerId|답변 고유 번호|

#### 2.4.1.3. Params
N/A

#### 2.4.1.4. Body
N/A

### 2.4.2. Response
N/A

### 2.4.3. Syntax

#### 2.4.3.1 Request Syntax

```json
DELETE https://mumulbo.com/api/v1/answers/{answerId}
```

#### 2.4.3.2. Response Syntax

```json
HTTP/1.1 204 No Content
```

<br/>

---

### 2.5. 답변 채택

|HTTP Method|URL|비고|
|------|---|---|
|PATCH|https://mumulbo.com/api/v1/|-|

### 2.5.1. Request

#### 2.5.1.1. Header
Authorization Bearer {JWT}

#### 2.5.1.2. Path Variables

|Key|Description|
|------|---|
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

```

#### 2.5.3.2. Response Syntax

```json

```

<br/>

---

### 2.6. 최근 댓글

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/answers/recent|-|

### 2.6.1. Request

#### 2.6.1.1. Header
Authorization Bearer {JWT}

#### 2.6.1.2. Path Variables
N/A

#### 2.6.1.3. Params
N/A

#### 2.6.1.4. Body
N/A

### 2.6.2. Response
#### 2.6.2.1 Header
N/A
#### 2.6.2.2 Body
|Key|Value|Description|
|------|---|---|
|answerId|1|답변 고유 번호|
|content|"답변이다냥"|답변 내용|
|author|"yuyeon.choe"|작성자 이름|
|status|"ACCEPTED"|답변 상태(ACCEPTED/NOT_ACCEPTED)|
|createdAt|"2025-05-06T23:26:04.436588"|생성 시간|
|updatedAt|"2025-05-07T23:26:04.436588"|수정 시간|

### 2.6.3. Syntax

#### 2.6.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/answers/recent
```

#### 2.6.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json
[
    {
        "answerId": 5,
        "content": "답변이다냥.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "ACCEPTED",
        "createdAt": "2025-05-11T23:26:04.436588",
        "updatedAt": "2025-05-11T23:26:04.436588"
    },
    {
        "answerId": 4,
        "content": "다른 답변이다냥2.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "NOT_ACCEPTED"
        "createdAt": "2025-05-10T23:26:04.436588",
        "updatedAt": "2025-05-10T23:26:04.436588"
    },
    {
        "answerId": 3,
        "content": "다른 답변이다냥.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "NOT_ACCEPTED"
        "createdAt": "2025-05-09T23:26:04.436588",
        "updatedAt": "2025-05-09T23:26:04.436588"
    },
    {
        "answerId": 2,
        "content": "다른 답변이다냥.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "NOT_ACCEPTED"
        "createdAt": "2025-05-08T23:26:04.436588",
        "updatedAt": "2025-05-08T23:26:04.436588"
    },
    {
        "answerId": 1,
        "content": "다른 답변이다냥.",
        "author": {
                "id": 123,
                "nickname": "yuyeonchoe",
                "picture": "images/profiles/01JWTWGSHTEM6QRN8VRMJN77YY"
            },
        "status": "NOT_ACCEPTED"
        "createdAt": "2025-05-07T23:26:04.436588",
        "updatedAt": "2025-05-07T23:26:04.436588"
    }
]

```

<br/>

---

### 2.7. 댓글 수

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/answers/count|-|

### 2.7.1. Request

#### 2.7.1.1. Header
Authorization Bearer {JWT}

#### 2.7.1.2. Path Variables

|Key|Description|
N/A

#### 2.7.1.3. Params
N/A

#### 2.7.1.4. Body


### 2.7.2. Response
#### 2.7.2.1 Header
N/A
#### 2.7.2.2 Body
|Key|Value|Description|
|------|---|---|
|totalCount|3|전체 답변 수|

### 2.7.3. Syntax

#### 2.7.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/answers/count

```

#### 2.7.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json
{
    "totalCount" : 3
}

```

<br/>

---

### 2.8. 가장 댓글이 많이 달린 질문

|HTTP Method|URL|비고|
|------|---|---|
||https://mumulbo.com/api/v1/|-|

### 2.8.1. Request

#### 2.8.1.1. Header
Authorization Bearer {JWT}

#### 2.8.1.2. Path Variables

|Key|Description|
|------|---|
|||

#### 2.8.1.3. Params
N/A

#### 2.8.1.4. Body
N/A

### 2.8.2. Response
N/A

### 2.8.3. Syntax

#### 2.8.3.1 Request Syntax

```json

```

#### 2.8.3.2. Response Syntax

```json

```

<br/>

---

### 2.9. 내가 작성한 댓글의 질문 목록

|HTTP Method|URL|비고|
|------|---|---|
||https://mumulbo.com/api/v1/|-|

### 2.9.1. Request

#### 2.9.1.1. Header
Authorization Bearer {JWT}

#### 2.9.1.2. Path Variables

|Key|Description|
|------|---|
|||

#### 2.9.1.3. Params
N/A

#### 2.9.1.4. Body
N/A

### 2.9.2. Response
N/A

### 2.9.3. Syntax

#### 2.9.3.1 Request Syntax

```json

```

#### 2.9.3.2. Response Syntax

```json

```



