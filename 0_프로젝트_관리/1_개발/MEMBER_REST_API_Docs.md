# 무물보 API 문서

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

| Error Code | Description            |
|------------|------------------------|
| SYSTEM-001 | 유효하지 않은 MethodArgument |
| AUTH-002   | 유효하지 않은 Token          |
| MEMBER-001 | 존재하지 않는 회원             |
| MEMBER-002 | 이미 가입한 회원              |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                           | 비고           |
|-------------|-----------------------------------------------|--------------|
| POST        | https://mumulbo.com/api/v1/auth/sign-up       | 회원가입         |
| POST        | https://mumulbo.com/api/v1/auth/sign-in       | 로그인          |
| POST        | https://mumulbo.com/api/v1/auth/refresh-token | 토큰 재발행       |
| GET         | https://mumulbo.com/api/v1/members/me         | 회원 정보 조회     |
| PUT         | -                                             | 회원 정보 수정     |
| DELETE      | -                                             | 회원탈퇴         |
| GET         | -                                             | 회원 프로필 사진 조회 |
| PUT         | -                                             | 회원 프로필 사진 수정 |
| DELETE      | -                                             | 회원 프로필 사진 삭제 |

---

### 2.1. 회원가입

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/auth/sign-up | -  |

### 2.1.1. Request

#### 2.1.1.1. Body

| Key      | Value                   | Required | Description | 비고                       |
|----------|-------------------------|----------|-------------|--------------------------|
| name     | 송준희                     | O        | 이름          | 최대 100글자까지 허용            |
| email    | joonhee.song@ahnlab.com | O        | 이메일         | 이메일 형식만 하용. 최대 254자까지 허용 |
| username | joonhee.song            | O        | 아이디         | 최대 20자까지 허용              |
| password | password                | O        | 비밀번호        | 최대 20자까지 허용              |

### 2.1.2. Response

#### 2.1.2.1. Body

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

``` json
POST https://mumulbo.com/api/v1/auth/sign-up  
Content-Type: application/json

{
    "name": "송준희",
    "email": "joonhee.song@ahnlab.com",
    "username": "joonhee.song",
    "password": "password"
}
```

#### 2.1.3.2. Response Syntax

``` json
요청이 성공한 경우 HttpStatus: 201
{
    "name": "송준희",
    "email": "joonhee.song@ahnlab.com",
    "username": "joonhee.song",
}

이미 가입된 회원인 경우 HttpStatus: 409
{
    "timestamp": "2025-04-09T12:41:18.126777888",
    "status": 409,
    "error": "MEMBER-002",
    "message": "이미 가입한 회원입니다."
}
```

---

### 2.2. 로그인

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/auth/sign-in | -  |

### 2.2.1. Request

#### 2.2.1.1. Body

| Key      | Value        | Required | Description | 비고          |
|----------|--------------|----------|-------------|-------------|
| username | joonhee.song | O        | 아이디         | 최대 20자까지 허용 |
| password | password     | O        | 비밀번호        | 최대 20자까지 허용 |

### 2.2.2. Response

#### 2.2.2.1. Body

| Key          | Value        | Description   |
|--------------|--------------|---------------|
| accessToken  | accessToken  | access token  |
| refreshToken | refreshToken | refresh token |

### 2.2.3. Syntax

#### 2.2.3.1 Request Syntax

``` json
POST https://mumulbo.com/api/v1/auth/sign-in  
Content-Type: application/json

{
    "username": "joonhee.song",
    "password": "password"
}
```

#### 2.2.3.2. Response Syntax

``` json
요청이 성공한 경우 HttpStatus: 201
{
    "accessToken": "accessToken",
    "refreshToken": "refreshToken"
}

일치하는 회언 정보가 없는 경우 HttpStatus: 409
{
    "timestamp": "2025-04-09T12:44:17.264602263",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.3. Token 재발행

| HTTP Method | URL                                           | 비고 |
|-------------|-----------------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/auth/refresh-token | -  |

### 2.3.1. Request

#### 2.3.1.1. Body

| Key          | Value        | Required | Description   |
|--------------|--------------|----------|---------------|
| refreshToken | refreshToken | O        | refresh token |

### 2.3.2. Response

#### 2.3.2.1. Body

| Key          | Value        | Description   |
|--------------|--------------|---------------|
| refreshToken | refreshToken | refresh token |
| accessToken  | accessToken  | access token  |

### 2.3.3. Syntax

#### 2.1.3.1 Request Syntax

``` json
POST https://mumulbo.com/api/v1/auth/refresh-token
Content-Type: application/json

{
    "refreshToken": "refreshToken"
}
```

#### 2.1.3.2. Response Syntax

``` json
토큰 재발급 성공한 경우 HttpStatus: 200
{
    "accessToken": "accessToken",
    "refreshToken": "refreshToken"
}

탈퇴한 회원이 요청한 경우 HttpStatus: 404
{
    "timestamp": "2025-04-09T12:44:17.264602263",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}

토큰이 유효하지 않은 경우 HttpStatus: 409
{
    "timestamp": "2025-04-09T22:40:04.295863",
    "status": 409,
    "error": "AUTH-002",
    "message": "유효하지 않은 토큰입니다."
}
```

---

### 2.4. 회원 정보 조회

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| GET         | https://mumulbo.com/api/v1/members/me | -  |

### 2.4.1. Request

#### 2.4.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

### 2.4.2. Response

#### 2.4.2.2. Body

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.4.3. Syntax

#### 2.4.3.2. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "username": "joonhee.song",
    "email": "joonhee.song@ahnlab.com",
}

토큰이 Header에 없는 경우 HttpStatus: 401
{
    "timestamp": "2025-04-09T22:49:09.192843",
    "status": 401,
    "error": "AUTH-001",
    "message": "인증되지 않은 요청입니다."
}

토큰이 유효하지 않은 경우 HttpStatus: 409
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 409,
    "error": "AUTH-002",
    "message": "유효하지 않은 토큰입니다."
}
```
