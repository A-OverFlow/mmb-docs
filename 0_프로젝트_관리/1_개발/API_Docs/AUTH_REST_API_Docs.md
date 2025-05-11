# 무물보 API 문서 - Auth Service

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

|HTTP Status Code|Custom Error Code|Description|응답 예시(Body)|
|------|---|---|---|
|**401** UNAUTHORIZED|`AUTH-4000`|인증되지 않음|<pre lang="json">{&#13;  "code": "AUTH-4000",&#13;  "message": null&#13;}</pre>|
|**401** UNAUTHORIZED|`AUTH-4001`|유효하지 않은 토큰|<pre lang="json">{&#13;  "code": "AUTH-4001",&#13;  "message": null&#13;}</pre>|

<br/><br/>

---

## 2. Auth Service

### 2.0. API 리스트

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/auth/signup|회원 가입|
|GET|https://mumulbo.com/api/v1/auth/validate|토큰 검증|
|POST|https://mumulbo.com/api/v1/auth/reissue|토큰 재발급|

<br/>

---

### 2.1. 회원 가입

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/auth/signup|-|

### 2.1.1. Request

|Key|Value|Required|Description|
|------|---|---|---|
|idToken|토큰|O|google Oauth idToken|


### 2.1.2. Response

#### 2.1.2.1. Body

|Key|Value|Description|
|------|---|---|
|accessToken|jwt 토큰|액세스 토큰|
|refreshToken|jwt 토큰|리프레시 토큰| 

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

```json
POST https://mumulbo.com/api/v1/auth/signup
```

#### 2.1.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5...",
    "refreshToken": "eyJhbGcI1NiIsInR56H9FNFUR..."
}
```

<br/>

---

### 2.2. 토큰 검증

|HTTP Method|URL|비고|
|------|---|---|
|GET|https://mumulbo.com/api/v1/auth/validate|-|

### 2.2.1. Request

#### 2.2.1.1. Header
|Key|Description|
|----|----|
|Authorization|eyJhbGciOiJIUzI1NiIsInR5...|

### 2.2.2. Response

#### 2.2.2.1. Body

|Key|Value|Description|
|------|---|---|
|valid|true|유효 여부|
|email|joonsub.lim@ahnlab.com|이메일|
|message|-|유효하지 않은 토큰인 경우 원인|

### 2.2.3. Syntax

#### 2.2.3.1 Request Syntax

```json
GET https://mumulbo.com/api/v1/auth/validate
```

#### 2.2.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "valid": false,
    "email": null,
    "message": "유효하지 않은 액세스 토큰입니다."
}
```

<br/>

---

### 2.3. 토큰 재발급

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/auth/reissue|-|

### 2.3.1. Request

#### 2.3.1.4. Body

|Key|Value|Required|Description|
|------|---|---|---|
|refreshToken|eyJhbGciOiJIUzI1NiIsInR5...|O|리프레시 토큰|

### 2.3.2. Response

#### 2.3.2.2. Body

|Key|Value|Description|
|------|---|---|
|accessToken|jwt 토큰|액세스 토큰|
|refreshToken|jwt 토큰|리프레시 토큰| 

### 2.3.3. Syntax

#### 2.3.3.1 Request Syntax

```json
POST https://mumulbo.com/api/v1/auth/reissue
Content-Type: application/json

{
    "subject": "제목입니다.",
    "content": "내용입니다.",
    "author": "heejaykong"
}
```

#### 2.3.3.2. Response Syntax

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5...",
    "refreshToken": "eyJhbGcI1NiIsInR56H9FNFUR..."
}
```

<br/>

---