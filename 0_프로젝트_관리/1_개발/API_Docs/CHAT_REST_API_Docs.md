# 무물보 API 문서 - Chat Service (개발용)

**Base URL**: `https://dev.mumulbo.com`

---

## 1. 공통

### 1.1. 에러 코드

> HTTP Error Code가 아닌 Custom Error Code 입니다.

| HTTP Status Code     | Custom Error Code | Description | 응답 예시(Body)                                |
|----------------------|-------------------|-------------|--------------------------------------------|
| **401** UNAUTHORIZED | `AUTH-4000`       | 인증되지 않음     | `{ "code": "AUTH-4000", "message": null }` |
| **401** UNAUTHORIZED | `AUTH-4001`       | 유효하지 않은 토큰  | `{ "code": "AUTH-4001", "message": null }` |

---

## 2. Chat Service

### 2.1. API 목록

| Method | URL                             | 설명                  | 인증 방식                                |
|--------|---------------------------------|---------------------|--------------------------------------|
| GET    | `/api/v1/chat/messages`         | 최근 채팅 목록 조회         | ❌ 인증 없음 (공개 채팅)                      |
| POST   | `/api/v1/chat/message`          | 채팅 메시지 저장           | ✅ `Authorization: Bearer <JWT>`      |
| WSS    | `wss://dev.mumulbo.com/ws/chat` | 실시간 채팅 WebSocket 연결 | ✅ `Authorization: Bearer <JWT>` (헤더) |

---

### 2.2. 채팅 목록 조회

- **URL**: `GET /api/v1/chat/messages`
- **설명**: 최근 12시간 이내 채팅 메시지를 조회합니다.
- **인증**: ❌ 없음 (공개 채팅)

#### ✅ 응답 예시

```json
[
  {
    "id": "6829afc038cba3206f992747",
    "senderName": "심명섭",
    "senderEmail": "test@test.com",
    "message": "안녕하세요",
    "type": "TEXT",
    "roomId": "main",
    "sentAt": "2025-05-18T10:00:32"
  }
]
```

---

### 2.3. 채팅 메시지 저장

- **URL**: `POST /api/v1/chat/message`
- **설명**: 채팅 메시지를 저장합니다.  
  운영 환경에서는 sender 정보는 JWT 토큰에서 추출됩니다.
- 웹소켓 기능 장애시 사용할수 있는 API 이며, 정상흐름에서는 필요없는 API입니다.
- 정상흐름시에는 실시간채팅시 DB에도 채팅데이터가 저장됨
- **인증**: ✅ `Authorization: Bearer <JWT>`

#### ✅ 요청 예시

```json
{
  "senderName": "심명섭",
  "senderEmail": "test@test.com",
  "message": "무물보 최고!",
  "type": "TEXT",
  "roomId": "main"
}
```

#### ✅ 응답 예시

```json
{
  "id": "6829bad521055a6a64b8c547",
  "senderName": "심명섭",
  "senderEmail": "test@test.com",
  "message": "무물보 최고!",
  "type": "TEXT",
  "roomId": "main",
  "sentAt": "2025-05-18T10:47:49"
}
```

---

### 2.4. 실시간 채팅 WebSocket

- **URL**: `wss://dev.mumulbo.com/ws/chat`
- **설명**: WebSocket을 통해 실시간 채팅을 송수신합니다.
- **인증**: ✅ JWT 토큰을 **헤더의 Authorization**에 넣어 전송
  ```
  Authorization: Bearer <JWT>
  ```

#### ✅ 연결 예시

WebSocket 라이브러리에서 헤더를 설정할 수 있어야 하며, 예시는 다음과 같습니다:

```javascript
//아래 5줄은 챗지피티가 알려준 소스이므로 검증필요.. 
new WebSocket("wss://dev.mumulbo.com/ws/chat", [], {
  headers: {
    Authorization: "Bearer valid-test-token"
  }
});
```

#### ✅ 요청 메시지 예시 (클라이언트 → 서버)

```json
{
  "type": "TEXT",
  "message": "안녕하세요"
}
```

#### ✅ 응답 메시지 예시 (서버 → 모든 클라이언트 브로드캐스트)

```json
{
  "id": null,
  "senderName": "심명섭",
  "senderEmail": "test@test.com",
  "message": "안녕하세요",
  "type": "TEXT",
  "roomId": "main",
  "sentAt": "2025-05-18T10:00:33"
}
```

---


![w14_ms_chat_test1.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test1.png)
![w14_ms_chat_test2.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test2.png)
![w14_ms_chat_test3.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test3.png)
postman wss프로토콜 사용시 header에 토큰넣는방법
![w14_ms_chat_test4.png](..%2F..%2F..%2F9_images%2Fw14_ms_chat_test4.png)