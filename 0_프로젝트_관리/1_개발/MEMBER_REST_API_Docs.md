# 무물보 API 문서

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

#### 1.1.1. 시스템 에러 코드

| Error Code | Description    |
|------------|----------------|
| SYSTEM-001 | 유효하지 않은 데이터 요청 |

#### 1.1.2. 회원 에러 코드

| Error Code | Description |
|------------|-------------|
| MEMBER-001 | 존재하지 않는 회원  |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                   | 비고       |
|-------------|---------------------------------------|----------|
| GET         | https://mumulbo.com/api/v1/members/me | 회원 정보 조회 |
| PUT         | https://mumulbo.com/api/v1/members/me | 회원 정보 수정 |

---

### 2.1. 회원 정보 조회

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| GET         | https://mumulbo.com/api/v1/members/me | -  |

### 2.1.1. Request

#### 2.1.1.1. Header

| Key       | Type | Value | Required | Description |
|-----------|------|-------|----------|-------------|
| X-USER-ID | Long | 1     | O        | 회원 아이디      |

### 2.1.2. Response

#### 2.1.2.1. Body (OAuth에서 제공하는 데이터 일부. 아직 미정)

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.1.3. Syntax

#### 2.1.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "username": "joonhee.song",
    "email": "joonhee.song@ahnlab.com",
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.2. 회원 정보 수정

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| PUT         | https://mumulbo.com/api/v1/members/me | -  |

### 2.2.1. Request

#### 2.2.1.1. Header

| Key       | Type | Value | Required | Description |
|-----------|------|-------|----------|-------------|
| X-USER-ID | Long | 1     | O        | 회원 아이디      |

#### 2.2.1.2. Body (OAuth에서 제공하는 데이터 일부. 아직 미정)

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.2.2. Response

#### 2.2.2.1. Body (OAuth에서 제공하는 데이터 일부. 아직 미정)

| Key      | Value                   | Description | 비고      |
|----------|-------------------------|-------------|---------|
| name     | 송준희                     | 이름          | 최대 100자 |
| email    | joonhee.song@ahnlab.com | 이메일         | 최대 254자 |
| username | joonhee.song            | 아이디         | 최대 20자  |

### 2.2.3. Syntax

#### 2.2.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "username": "joonhee.song",
    "email": "joonhee.song@ahnlab.com",
}

잘못된 데이터로 요청한 경우 HttpStatus: 400
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 400,
    "error": "SYSTEM-001",
    "message": "-"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}

```
