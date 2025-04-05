# 무물보 API 문서

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

| Error Code | Description  |
|------------|--------------|
| 1000       | 잘못된 argument |
| 1001       | 데이터 없음       |
| 1002       | 알수없는 에러      |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                                  | 비고           |
|-------------|------------------------------------------------------|--------------|
| POST        | https://mumulbo.com/api/v1/auth/sign-up              | 회원가입         |
| POST        | https://mumulbo.com/api/v1/auth/sign-in              | 로그인          |
| GET         | https://mumulbo.com/api/v1/member/{memberId}         | 회원 정보 조회     |
| PUT         | https://mumulbo.com/api/v1/member/{memberId}         | 회원 정보 수정     |
| DELETE      | https://mumulbo.com/api/v1/member/{memberId}         | 회원탈퇴         |
| GET         | https://mumulbo.com/api/v1/member/{memberId}/profile | 회원 프로필 사진 조회 |
| PUT         | https://mumulbo.com/api/v1/member/{memberId}/profile | 회원 프로필 사진 수정 |
| DELETE      | https://mumulbo.com/api/v1/member/{memberId}/profile | 회원 프로필 사진 삭제 |

---

### 2.1. 회원가입

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/auth/sign-up | -  |

### 2.1.1. Request

#### 2.1.1.1. Body

| Key      | Value                   | Required | Description |
|----------|-------------------------|----------|-------------|
| name     | 송준희                     | O        | 이름          |
| email    | joonhee.song@ahnlab.com | O        | 이메일         |
| username | joonhee.song            | O        | 아이디         |
| password | password                | O        | 비밀번호        |

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
{
    "name": "송준희",
    "email": "joonhee.song@ahnlab.com",
    "username": "joonhee.song",
}
```

---

### 2.2. 로그인

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/auth/sign-in | -  |

### 2.2.1. Request

#### 2.2.1.1. Body

| Key      | Value        | Required | Description |
|----------|--------------|----------|-------------|
| username | joonhee.song | O        | 아이디         |
| password | password     | O        | 비밀번호        |

### 2.2.2. Response

#### 2.2.2.1. Body

| Key          | Value | Description   |
|--------------|-------|---------------|
| accessToken  | ---   | access token  |
| refreshToken | ---   | refresh token |

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
{
    "accessToken": "String",
    "refreshToken": "String"
}
```
