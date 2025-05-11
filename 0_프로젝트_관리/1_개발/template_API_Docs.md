> 아래 목차는 예시이며 작성자 임의로 목록을 추가하거나 수정해서 사용할 수 있습니다.

# Sample Service API Docs

## POST /api/v1/auth/login

* **API 이름**: 사용자 로그인
* **요청 URL**: `POST /api/v1/auth/login`
* **설명**: 사용자가 이메일과 비밀번호로 로그인하는 API입니다.
* **인증 필요 여부**: ❌ (비로그인 상태에서 접근 가능)

### 요청(Request)

#### HTTP Method

- POST

#### Headers

| 이름           | 타입                 | 필수 여부 | 설명           |
|--------------|--------------------|-------|--------------|
| Content-Type | `application/json` | ✅     | 요청 본문의 타입 지정 |

#### Request Body (JSON)

```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

| 필드명      | 타입     | 필수 여부 | 설명         |
|----------|--------|-------|------------|
| email    | string | ✅     | 사용자 이메일 주소 |
| password | string | ✅     | 사용자 비밀번호   |

### 응답(Response)

#### 성공 (200 OK)

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR...",
  "expiresIn": 3600
}
```

| 필드명          | 타입     | 설명                 |
|--------------|--------|--------------------|
| accessToken  | string | 인증에 사용할 JWT 토큰     |
| refreshToken | string | 토큰 갱신에 사용할 리프레시 토큰 |
| expiresIn    | number | 토큰 만료 시간 (초 단위)    |

#### 실패 (예시: 401 Unauthorized)

```json
{
  "status": 401,
  "message": "이메일 또는 비밀번호가 올바르지 않습니다."
}
```

| 필드명     | 타입     | 설명         |
|---------|--------|------------|
| status  | number | HTTP 상태 코드 |
| message | string | 에러 메시지     |

### 참고 사항

* 비밀번호는 최소 8자 이상이어야 하며, 영문과 숫자를 포함해야 합니다.
* 인증 성공 시 반환되는 토큰은 이후 API 호출 시 `Authorization: Bearer <accessToken>` 형식으로 사용합니다.