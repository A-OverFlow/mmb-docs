<img width="939" alt="스크린샷 2025-06-21 오후 9 18 43" src="https://github.com/user-attachments/assets/d2203931-d1aa-4ba4-83e2-9bc0e07ad3f5" /># 무물보 API 문서 - Chat Service (개발용)

**Base URL**: `https://dev.mumulbo.com`

---

## 1. 공통

### 1.1. 에러 코드

> HTTP Error Code 기준의 일반적인 응답입니다. (커스텀 코드 없음)

| HTTP Status Code     | Description    | 예시 응답 Body          |
|----------------------|----------------|---------------------|
| **401** UNAUTHORIZED | 인증 실패 또는 토큰 누락 | 없음 또는 빈 JSON (`{}`) |

---

## 2. Chat Service

### 2.1. API 목록

| Method | URL                                    | 설명                  | 인증 방식                                |
|--------|----------------------------------------|---------------------|--------------------------------------|
| GET    | `/api/v1/chat/messages`                | 최근 채팅 목록 조회         | ❌ 인증 없음 (공개 채팅)                      |
| POST   | `/api/v1/chat/message`                 | 채팅 메시지 저장           | ✅ `Authorization: Bearer <JWT>`      |
| WSS    | `wss://dev.mumulbo.com/api/v1/ws/chat` | 실시간 채팅 WebSocket 연결 | URL Query String 에 토큰을 포함시켜 전달

WSS 요청 예시

wss://dev.mumulbo.com/api/v1/ws/chat?token=eyJhbdddddd.eyJzdWIiOiI0ffffffffNzB9.Pxh6UVIJneF2ta5Nv0
---

### 2.2. 채팅 목록 조회

* **URL**: `GET /api/v1/chat/messages`
* **설명**: 최근 12시간 이내 채팅 메시지를 조회합니다.
  `WHISPER` 타입(귓속말)은 포함되지 않습니다.
* **인증**: ❌ 없음 (공개 채팅)

#### ✅ 응답 예시

```json
[
  {
    "id": "684e2ba89371043b93cd792d",
    "senderName": "",
    "senderEmail": "leo@mumulbo.com",
    "message": "전체메시지~",
    "type": "TEXT",
    "roomId": "main",
    "recipientUserId": null,
    "sentAt": "2025-06-15T02:10:48",
    "connectedUserList": null
  },
  {
    "id": "684e22ab2520487bbfbc967d",
    "senderName": "심명섭",
    "senderEmail": "test@test.com",
    "message": "무물보 최고!",
    "type": "TEXT",
    "roomId": "main",
    "recipientUserId": null,
    "sentAt": "2025-06-15T01:32:27",
    "connectedUserList": null
  }
]
```

---

### 2.3. 채팅 메시지 저장

* **URL**: `POST /api/v1/chat/message`
* **설명**: 채팅 메시지를 저장합니다.
  운영 환경에서는 sender 정보는 JWT 토큰에서 추출됩니다.
  웹소켓 기능 장애 시 사용할 수 있는 API이며, 정상 흐름에서는 필요 없습니다.
  실시간 채팅 중에는 메시지가 WebSocket을 통해 MongoDB에 자동 저장됩니다.
* **인증**: ✅ `Authorization: Bearer <JWT>`

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
  "id": "684e3ecb0ed567260d8f5a22",
  "senderName": "심명섭",
  "senderEmail": "test@test.com",
  "message": "무물보 최고!",
  "type": "TEXT",
  "roomId": "main",
  "recipientUserId": null,
  "sentAt": "2025-06-15T03:32:27",
  "connectedUserList": null
}
```

---

### 2.4. 실시간 채팅 WebSocket

* **URL**: `wss://dev.mumulbo.com/api/v1/ws/chat`
* **설명**: WebSocket을 통해 실시간 채팅을 송수신합니다.
* **인증**: ✅ JWT 토큰을 **헤더의 Authorization**에 넣어 전송

  ```
  Authorization: Bearer <JWT>
  ```

#### ✅ 연결 예시

```javascript
// 예시 - 실제 사용 시 WebSocket 클라이언트 라이브러리 필요
const token = "valid-test-token";
const socket = new WebSocket(`wss://dev.mumulbo.com/api/v1/ws/chat?token=${encodeURIComponent(token)}`);
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

### 2.5. 귓속말 (Whisper) 메시지 전송 및 실패 처리

* **설명**: 특정 사용자에게만 전달되는 귓속말 메시지를 WebSocket을 통해 전송합니다. 귓속말은 브로드캐스트되지 않으며, 송신자와 수신자만 해당 메시지를 수신합니다.
  수신자가 접속 중이 아니거나 본인에게 보낸 경우, 실패 메시지를 전송합니다.
* **전송 방식**: WebSocket
* **메시지 타입**:

    * 전송: `WHISPER`
    * 실패 시 응답: `WHISPER_FAILED`
* **인증**: ✅ `Authorization: Bearer <JWT>`

#### ✅ 귓속말 요청 메시지 예시 (클라이언트 → 서버)

```json
{
  "type": "WHISPER",
  "message": "귓속말 내용입니다",
  "recipientUserId": 2
}
```

#### ✅ 성공 응답 예시 (서버 → 송신자와 수신자 모두에게 전송)

```json
{
  "type": "WHISPER",
  "senderName": "심명섭",
  "senderEmail": "test@test.com",
  "message": "귓속말 내용입니다",
  "recipientUserId": 2,
  "sentAt": "2025-06-15T16:40:00"
}
```

#### ❌ 실패 응답 예시 (서버 → 송신자에게만 전송)

```json
{
  "type": "WHISPER_FAILED",
  "message": "귓속말 전송 실패: 상대가 접속 중이 아닙니다.",
  "recipientUserId": 2,
  "sentAt": "2025-06-15T16:40:00"
}
```

### 2.6. 현재 접속자 목록 실시간 브로드캐스트 (WebSocket)

* **설명**: 사용자가 WebSocket에 접속하거나 연결이 종료되면, 서버는 접속 중인 전체 사용자 목록을 `USER_LIST_UPDATE` 메시지로 브로드캐스트합니다.
  이 메시지를 수신한 클라이언트는 실시간으로 사용자 목록 UI를 갱신해야 합니다.
* **전송 방식**: WebSocket
* **메시지 타입**: `USER_LIST_UPDATE`
* **인증**: ✅ `Authorization: Bearer <JWT>`

#### ✅ 수신 메시지 예시 (서버 → 전체 사용자)

```json
{
  "type": "USER_LIST_UPDATE",
  "connectedUserList": [
    1,
    2,
    3
  ],
  "sentAt": "2025-06-15T16:30:00"
}json
{
  "type": "USER_LIST_UPDATE",
  "connectedUserList": [
    1,
    2,
    3
  ],
  "sentAt": "2025-06-15T16:30:00"
}
```

![w14_ms_chat_test1.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test1.png)
![w14_ms_chat_test2.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test2.png)
![w14_ms_chat_test3.png](..%2F..%2F..%2F9_images%2Fw11_ms_chat_test3.png)
postman wss프로토콜 사용시 header에 토큰넣는방법
![w14_ms_chat_test4.png](..%2F..%2F..%2F9_images%2Fw14_ms_chat_test4.png)
