# 무물보 API 문서

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

#### 1.1.1. 시스템 에러 코드

| Error Code | Description |
|------------|-------------|
| -          | -           |

#### 1.1.2. 회원 에러 코드

| Error Code | Description |
|------------|-------------|
| MEMBER-001 | 존재하지 않는 회원  |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                   | Description   | 비고     |
|-------------|---------------------------------------|---------------|--------|
| POST        | https://mumulbo.com/api/v1/members    | 회원 정보 저장 및 조회 | 내부 통신용 |
| GET         | https://mumulbo.com/api/v1/members/me | 회원 정보 조회      | -      |
| PUT         | https://mumulbo.com/api/v1/members/me | 회원 정보 수정      | -      |
| DELETE      | https://mumulbo.com/api/v1/members/me | 회원 정보 삭제      | -      |

---

### 2.1. 회원 정보 저장 및 조회

| HTTP Method | URL                                | 비고                          |
|-------------|------------------------------------|-----------------------------|
| POST        | https://mumulbo.com/api/v1/members | 회원 정보 저장(가입되지 않은 사용자만) 및 조회 |

### 2.1.1. Request

#### 2.1.1.1. Body

| Key        | Value                 | Description           |
|------------|-----------------------|-----------------------|
| provider   | GOOGLE                | Authentication Server |
| providerId | 012345678901234567890 | idToken               |
| name       | 송준희                   | 이름                    |
| email      | mike.urssu@gmail.com  | 이메일                   |

### 2.1.2. Response

#### 2.1.2.1. Body

| Key | Value | Description |
|-----|-------|-------------|
| id  | 1     | 회원 id       |

### 2.1.3. Syntax

#### 2.1.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "id": 1
}
```

---

### 2.2. 회원 정보 조회

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| GET         | https://mumulbo.com/api/v1/members/me | -  |

### 2.2.1. Request

#### 2.2.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

### 2.2.2. Response

#### 2.2.2.1. Body

| Key   | Value                | Description |
|-------|----------------------|-------------|
| name  | 송준희                  | 이름          |
| email | mike.urssu@gmail.com | 이메일         |

### 2.2.3. Syntax

#### 2.2.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "email": "mike.urssu@gmail.com"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.3. 회원 정보 수정

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| PUT         | https://mumulbo.com/api/v1/members/me | -  |

### 2.3.1. Request

#### 2.3.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

#### 2.3.1.2. Body

| Key   | Value                 | Description |
|-------|-----------------------|-------------|
| email | mike.urssu2@gmail.com | 이메일         |

### 2.3.2. Response

#### 2.3.2.1. Body

| Key   | Value                 | Description |
|-------|-----------------------|-------------|
| email | mike.urssu2@gmail.com | 이메일         |

### 2.3.3. Syntax

#### 2.3.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "email": "mike.urssu2@gmail.com"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.4. 회원 탈퇴

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| DELETE      | https://mumulbo.com/api/v1/members/me | -  |

### 2.4.1. Request

#### 2.4.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

### 2.4.2. Syntax

#### 2.4.2.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 204

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```
