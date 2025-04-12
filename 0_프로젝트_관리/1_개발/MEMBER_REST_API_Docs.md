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
| MEMBER-002 | 이미 존재하는 회원  |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                      | 비고       |
|-------------|------------------------------------------|----------|
| POST        | https://mumulbo.com/api/v1/members/check | 회원 여부 확인 |
| POST        | https://mumulbo.com/api/v1/members       | 회원가입     |
| GET         | https://mumulbo.com/api/v1/members/{id}  | 회원 정보 조회 |
| PUT         | https://mumulbo.com/api/v1/members/{id}  | 회원 정보 수정 |
| DELETE      | https://mumulbo.com/api/v1/members/{id}  | 회원 탈퇴    |

---

### 2.1. 회원 여부 확인

| HTTP Method | URL                                      | 비고                       |
|-------------|------------------------------------------|--------------------------|
| POST        | https://mumulbo.com/api/v1/members/check | 로그인 시 일치하는 회원 정보가 있는지 확인 |

### 2.1.1. Request

#### 2.1.1.1. Body (아직 미정)

| Key   | Value                   | Description |
|-------|-------------------------|-------------|
| email | joonhee.song@ahnlab.com | 이메일         |

### 2.1.2. Response

#### 2.1.2.1. Body (아직 미정)

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

일치하는 회원이 없는 경우: HttpStatus: 404
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.2. 회원가입

| HTTP Method | URL                                | 비고 |
|-------------|------------------------------------|----|
| POST        | https://mumulbo.com/api/v1/members | -  |

### 2.2.1. Request

#### 2.2.1.1. Body (아직 미정)

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.2.2. Response

#### 2.2.2.1. Body (아직 미정)

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| id       | 1                       | 회원 id       |
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.2.4. Syntax

#### 2.2.4.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 201
{
    "id": 1,
    "name": "송준희",
    "username": "joonhee.song",
    "email": "joonhee.song@ahnlab.com",
}

이미 가입한 회원인 경우 HttpStatus: 409
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 409,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.3. 회원 정보 조회

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| GET         | https://mumulbo.com/api/v1/members/{id} | -  |

### 2.3.1. Request

#### 2.3.1.1. Path Variables

| Key | Value | Description |
|-----|-------|-------------|
| id  | 1     | 회원 아이디      |

### 2.3.2. Response

#### 2.3.2.1. Body (아직 미정)

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | joonhee.song@ahnlab.com | 이메일         |
| username | joonhee.song            | 아이디         |

### 2.3.3. Syntax

#### 2.3.3.1. Response Syntax

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

### 2.4. 회원 정보 수정

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| PUT         | https://mumulbo.com/api/v1/members/{id} | -  |

### 2.4.1. Request

#### 2.4.1.1. Path Variables

| Key | Value | Description |
|-----|-------|-------------|
| id  | 1     | 회원 아이디      |

#### 2.4.1.2. Body (아직 미정)

| Key      | Value                    | Description |
|----------|--------------------------|-------------|
| name     | 송준희2                     | 이름          |
| email    | joonhee.song2@ahnlab.com | 이메일         |
| username | joonhee.song2            | 아이디         |

### 2.4.2. Response

#### 2.4.2.1. Body (아직 미정)

| Key      | Value                    | Description | 비고      |
|----------|--------------------------|-------------|---------|
| name     | 송준희2                     | 이름          | 최대 100자 |
| email    | joonhee.song2@ahnlab.com | 이메일         | 최대 254자 |
| username | joonhee.song2            | 아이디         | 최대 20자  |

### 2.4.3. Syntax

#### 2.4.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희2",
    "username": "joonhee.song2",
    "email": "joonhee.song2@ahnlab.com",
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

### 2.5. 회원 탈퇴

| HTTP Method | URL                                     | 비고 |
|-------------|-----------------------------------------|----|
| DELETE      | https://mumulbo.com/api/v1/members/{id} | -  |

### 2.5.1. Request

#### 2.5.1.1. Path Variables

| Key | Value | Description |
|-----|-------|-------------|
| id  | 1     | 회원 아이디      |

### 2.5.2. Syntax

#### 2.5.2.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 204

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "timestamp": "2025-04-09T22:48:42.099408",
    "status": 404,
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}

```
