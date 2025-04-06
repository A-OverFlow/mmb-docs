# REST (Representational State Transfer) 란?

- 웹 아키텍처 스타일
- 상태를 표현(Representational), 그리고 전달(Transfer)
- 웹 표준을 최대로 활용하여 클라이언트-서버 간 통신을 단순하고 일관되게 하자는 철학

## 핵심 개념

- 자원(Resource) : URI로 식별 (예: /users/1)
- 표현(Representation) : 자원의 상태를 표현하는 형태 (보통 JSON)
- 상태 전달(State Transfer) : 클라이언트와 서버가 자원의 상태를 주고받는 구조

"자원을 URI로 정의" 하고, "자원의 상태를 HTTP 메서드로 전달" 한다는 개념입니다.

## 다음을 만족해야 RESTful 하다고 할 수 있다

1. HTTP 메서드를 의미에 맞게 사용 (GET, POST, PUT, PATCH, DELETE)
2. 엔드포인트는 명사 + 복수형으로
3. **상태 코드를 정확하게 구분해서 반환 (HTTP Response Code)**
4. JSON 형식으로 응답 통일
5. URI 버전 관리

# HTTP 메서드 사용 규칙

| 행위 | HTTP 메서드 | 설명                    |
|----|----------|-----------------------|
| 조회 | `GET`    | 리소스 조회 (데이터를 변경하지 않음) |
| 생성 | `POST`   | 새로운 리소스 생성            |
| 수정 | `PUT`    | 전체 리소스 수정 (대체)        |
| 수정 | `PATCH`  | 일부 필드만 수정             |
| 삭제 | `DELETE` | 리소스 삭제                |

# 엔드포인트(Endpoint) 설계 규칙

- 항상 **복수형 명사** 사용: `users`, `orders`, `products`
  ```http
  GET /users        // 사용자 목록 조회
  GET /users/1      // 특정 사용자 조회
  POST /users       // 사용자 생성
  PUT /users/1      // 사용자 전체 수정
  PATCH /users/1    // 사용자 일부 수정
  DELETE /users/1   // 사용자 삭제
  ```

- 중첩 리소스는 관계 표현에 사용
  ```http
  GET /users/1/orders        // 사용자 1의 주문 목록
  GET /users/1/orders/99     // 사용자 1의 주문 중 ID가 99인 주문
  ```

# 쿼리 파라미터 사용 (Filtering, Pagination, Sorting)

- 필터링: `/products?category=shoes&color=black`
- 페이징: `/products?page=2&limit=10`
- 정렬: `/products?sort=price,asc`

# 응답 코드 사용

자주 사용하는 응답 코드

| 상태 코드                       | 설명               |
|-----------------------------|------------------|
| `200 OK`                    | 요청 성공 (조회/수정 등)  |
| `201 Created`               | 리소스 생성 성공        |
| `204 No Content`            | 삭제 성공 (응답 본문 없음) |
| `400 Bad Request`           | 잘못된 요청           |
| `401 Unauthorized`          | 인증 실패            |
| `403 Forbidden`             | 권한 없음            |
| `404 Not Found`             | 리소스를 찾을 수 없음     |
| `500 Internal Server Error` | 서버 오류            |

## JSON 응답 구조 예시 (일관성 중요)

```json
{
  "data": {
    "id": 1,
    "name": "홍길동",
    "email": "hong@example.com"
  },
  "meta": {
    "timestamp": "2025-04-07T12:00:00Z"
  }
}
```

## 에러 응답 구조

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "해당 사용자를 찾을 수 없습니다."
  }
}
```

- code
    - 도메인 에러 코드 (클라이언트에 명확한 식별자 제공)
    - 같은 에러 코드라도 다양한 이유로 발생할 수 있기 때문
    - 정확한 에러 발생 이유를 구분하기 위하여 사용

# 버전 관리

- URI 버전: 추천 방식
  ```http
  GET /api/v1/users
  ```

---

# 예외 처리 및 메시지 일관성

- 클라이언트가 예외 원인을 명확히 알 수 있게 처리
- 메시지는 **국제화(i18n)** 고려