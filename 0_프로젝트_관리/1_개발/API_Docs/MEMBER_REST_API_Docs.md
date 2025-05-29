# 무물보 회원 서비스 API 문서

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

#### 1.1.2. 회원 에러 코드

| Error Code  | Description         |
|-------------|---------------------|
| PROFILE-001 | 이미지 확장자를 지원하지 않는 경우 |

---

## 2. Member Service

### 2.0. API 리스트

| HTTP Method | URL                                                   | Description | 비고     |
|-------------|-------------------------------------------------------|-------------|--------|
| POST        | https://mumulbo.com/api/v1/members                    | 회원 정보 저장    | 내부 통신용 |
| GET         | https://mumulbo.com/api/v1/members/me                 | 회원 정보 조회    | -      |
| DELETE      | https://mumulbo.com/api/v1/members/me                 | 회원 정보 삭제    | -      |
| GET         | https://mumulbo.com/api/v1/members/me/profile         | 프로필 조회      | -      |
| PUT         | https://mumulbo.com/api/v1/members/me/profile/picture | 프로필 이미지 수정  | -      |
| PATCH       | https://mumulbo.com/api/v1/members/me/profile/info    | 프로필 정보 수정   | -      |

---

### 2.1. 회원 정보 저장 및 조회

| HTTP Method | URL                                | 비고                     |
|-------------|------------------------------------|------------------------|
| POST        | https://mumulbo.com/api/v1/members | 회원 정보 저장(가입되지 않은 사용자만) |

### 2.1.1. Request

#### 2.1.1.1. Body

| Key        | Value                                              | Description             |
|------------|----------------------------------------------------|-------------------------|
| provider   | GOOGLE                                             | Authentication Server   |
| providerId | 012345678901234567890                              | idToken                 |
| name       | 송준희                                                | 이름                      |
| email      | mike.urssu@gmail.com                               | 이메일                     |
| picture    | https://lh3.googleusercontent.com/a/abcdefg1234567 | `Google`에서 제공하는 프로필 url |

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

| Key      | Value                   | Description |
|----------|-------------------------|-------------|
| name     | 송준희                     | 이름          |
| email    | mike.urssu@gmail.com    | 이메일         |
| nickname | nickname                | 닉네임         |
| picture  | images/profiles/abcdefg | 프로필 url     |

### 2.2.3. Syntax

#### 2.2.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "email": "mike.urssu@gmail.com",
    "nickname": "nickname",
    "picture": "images/profiles/abcdefg"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.3. 회원 탈퇴

| HTTP Method | URL                                   | 비고 |
|-------------|---------------------------------------|----|
| DELETE      | https://mumulbo.com/api/v1/members/me | -  |

### 2.3.1. Request

#### 2.3.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

### 2.3.2. Syntax

#### 2.3.2.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 204

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.4. 프로필 조회

| HTTP Method | URL                                           | 비고 |
|-------------|-----------------------------------------------|----|
| GET         | https://mumulbo.com/api/v1/members/me/profile | -  |

### 2.4.1. Request

#### 2.4.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

### 2.4.2. Response

#### 2.4.2.1. Body

| Key          | Value                         | Description |
|--------------|-------------------------------|-------------|
| name         | 송준희                           | 이름          |
| email        | mike.urssu@gmail.com          | 이메일         |
| nickname     | nickname                      | 닉네임         |
| picture      | images/profiles/abcdefg       | 프로필 url     |
| introduction | 안녕하세요! 뚱땅뚱땅입니다!               | 자기소개 문구     |
| website      | https://github.com/mike-urssu | 개인 website  |

### 2.4.3. Syntax

#### 2.4.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "name": "송준희",
    "email": "mike.urssu@gmail.com",
    "nickname": "nickname",
    "picture": "images/profiles/abcdefg",
    "introduction": "안녕하세요! 뚱땅뚱땅입니다!",
    "website": "https://github.com/mike-urssu"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```

---

### 2.5. 프로필 이미지 수정

| HTTP Method | URL                                                   | 비고 |
|-------------|-------------------------------------------------------|----|
| PUT         | https://mumulbo.com/api/v1/members/me/profile/picture | -  |

### 2.5.1. Request

#### 2.5.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

#### 2.5.1.2. Body (⚠️ `form-data`로 요청)

| Key  | Value    | Description |
|------|----------|-------------|
| file | `이미지 파일` | 프로필 url     |

### 2.5.2. Response

#### 2.5.2.1. Body

| Key     | Value                   | Description |
|---------|-------------------------|-------------|
| picture | images/profiles/hijklmn | 프로필 url     |

### 2.5.3. Syntax

#### 2.5.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "picture": "images/profiles/hijklmn"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}

이미지 파일이 아닌 경우 HttpStatus: 415
{
    "error": "PROFILE-001",
    "message": "지원하지 않는 파일입니다."
}
```

---

### 2.6. 프로필 정보 수정

| HTTP Method | URL                                                | 비고 |
|-------------|----------------------------------------------------|----|
| PATCH       | https://mumulbo.com/api/v1/members/me/profile/info | -  |

### 2.6.1. Request

#### 2.6.1.1. Header

| Key           | Value        | Required | Description |
|---------------|--------------|----------|-------------|
| Authorization | Bearer <JWT> | O        | 토큰          |

#### 2.6.1.2. Body

| Key          | Value                         | Description | Required | 비고                  |
|--------------|-------------------------------|-------------|----------|---------------------|
| nickname     | new nickname                  | 닉네임         | X        | `null` 값을 보내면 갱신 안함 |
| introduction | 안녕하세요! 뚱땅뚱땅입니다!               | 자기소개 문구     | X        | 요청에 포함된 경우에만 갱신     |
| website      | https://github.com/mike-urssu | 개인 website  | X        | 요청에 포함된 경우에만 갱신     |

### 2.6.2. Response

#### 2.6.2.1. Body

| Key          | Value                         | Description |
|--------------|-------------------------------|-------------|
| nickname     | new nickname                  | 닉네임         |
| introduction | 안녕하세요! 뚱땅뚱땅입니다!               | 자기소개 문구     |
| website      | https://github.com/mike-urssu | 개인 website  |

### 2.6.3. Syntax

#### 2.6.3.1. Response Syntax

``` json
요청에 성공한 경우: HttpStatus: 200
{
    "nickname": "new nickname",
    "introduction": "안녕하세요! 뚱땅뚱땅입니다!",
    "website": "https://github.com/mike-urssu"
}

회원이 존재하지 않는 경우 HttpStatus: 404
{
    "error": "MEMBER-001",
    "message": "존재하지 않는 회원입니다."
}
```
