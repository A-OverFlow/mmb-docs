# 📄 채팅 서비스 SRS (Software Requirements Specification)

---

## 1. 개요

### 1.1 문서 목적

- 이 문서는 “무물보” 서비스에 추가되는 “오픈 채팅 기능”의 구현을 위해 명세를 정의합니다.

### 1.2 시스템 요약

- 프론트엔드(React)에서 API Gateway(Spring Cloud Gateway)를 거쳐, 채팅 서버(ChatService, Kotlin 기반)와 통신
- WebSocket을 통한 실시간 메시지 처리 + MongoDB에 로그 저장
- Redis Pub/Sub을 통한 서버 내부 브로드캐스트
- 초기기능은 실시간채팅, 초기채팅 불러오기 기능을 지원하며 추후 귓속말, 이모지등을 지원예정

### 1.3 용어 정의

- **TEXT**: 일반 텍스트 메시지
- **EMOJI**: 이모지 전송 (향후 지원)
- **WHISPER**: 귓속말 (향후 지원)
- **SYSTEM**: 시스템 알림 메시지 (예: 입장, 퇴장)

---

## 2. 시스템 개요

### 2.1 전체 아키텍처

```
브라우저
  ↓
API Gateway
  ↓
ChatService (WebSocket + REST)
  ├─ Redis (Pub/Sub)
  └─ MongoDB (채팅 로그 저장)
```

- WebSocket 연결 시: Gateway는 Upgrade 헤더 감지 후 라우팅만 수행, 인증은 ChatService에서 직접 수행
- REST 요청 시: Gateway의 JWT 인증 필터가 작동함

### 2.2 흐름 정리

#### WebSocket (실시간 채팅)

1. `/chat` 라우팅 → Gateway는 wss 프로토콜이면 jwt 인증없이 라우팅만 함
2. ChatService에서 JWT 인증 수행
3. 메시지 Redis에 `pub`
4. 서버 내부에서 `sub` → WebSocket 연결된 세션에 브로드캐스트
5. 메시지를 MongoDB에 저장

#### REST (메시지 조회)

1. `GET /chat/messages` API 호출 (HTTP)
2. `POST /chat/message` API 호출 (HTTP) (웹소켓 연결안될경우 채팅로그 적재용)
3. Gateway의 JWT 인증 필터 통과
4. ChatService에서 MongoDB 조회 후 JSON 응답

---

## 3. 기능 요구사항

### 3.1 기본 기능

- 채팅 메시지 전송 및 수신 (TEXT)
- WebSocket 기반 실시간 브로드캐스트
- 채팅 기록 저장 및 조회 (MongoDB)
- 메시지 조회 시 최대 12시간까지만 반환
- WebSocket이 안 될 경우를 대비한 fallback용 REST 전송 API

### 3.2 향후 확장 기능

- 귓속말 기능 (receiverId 포함)
- 이모지 전송 (이모지 코드 렌더링)

---

## 4. 비기능 요구사항

- JWT 인증 기반 사용자 식별
- WebSocket 인증은 ChatService에서 직접 수행 (Query 파라미터 등)
- 메시지는 MongoDB에 저장, TTL 인덱스 사용 (12시간 보존)
- Redis Pub/Sub을 통한 내부 메시지 전달 처리
- WebSocket 연결은 채팅창을 클릭했을 때 수동으로 맺음 (성능 최적화)

---

## 5. 시스템 아키텍처

### 5.1 인증 흐름

- REST: 브라우저 → API Gateway (JWT 필터 통과) → ChatService → MongoDB
- WebSocket: 브라우저 → Gateway (경로 라우팅만 함, 필터 안 탐) → ChatService (내부에서 JWT 직접 검증)

### 5.2 Gateway 필터 분기 로직 (WebSocket용)

- `Upgrade: websocket` 헤더 확인하여 필터 건너뜀
- HTTP 요청일 경우 기존 필터로 인증 수행

---

## 6. 데이터 설계

### 6.1 ERD: ChatMessage

```
ChatMessage
-------------
- id: String (UUID)
- senderId: String
- senderNickname: String
- message: String
- sentAt: DateTime (ISO 8601)
- type: ENUM (TEXT, EMOJI, SYSTEM)
- roomId: String (기본 오픈채팅방은 fixed value)
```

→ MongoDB에 `chat_messages` 컬렉션으로 저장  
→ TTL 인덱스로 12시간 경과 후 자동 삭제

```js
// MongoDB TTL 설정
db.chat_messages.createIndex({"sentAt": 1}, {expireAfterSeconds: 43200}) // 12시간
```

---

## 7. API 명세 요약

### 7.1 WebSocket 연결

- URI: `wss://dev.mumulbo.com/ws/chat?token=xxx`
- 연결 시점: 채팅 버튼 클릭 시 수동 연결
- 연결 후 JWT 검증: ChatService 내부에서 처리

### 7.2 메시지 전송 (WebSocket)

```json
// 클라이언트 → 서버
{
  "type": "TEXT",
  "message": "안녕하세요"
}

// 서버 → 클라이언트
{
  "senderId": "user1",
  "senderNickname": "홍길동",
  "type": "TEXT",
  "message": "안녕하세요",
  "sentAt": "2025-05-01T09:00:00Z"
}
```

### 7.3 메시지 조회 (REST)

- GET `/chat/messages`
- 응답: 최근 12시간 메시지 배열 (최신순 정렬)
- 인증: API Gateway에서 처리

---

## 8. 테스트 도구

### WebSocket 테스트 방법

1. **websocat (CLI)**

```bash
websocat "wss://dev.mumulbo.com/ws/chat?token=xxx"
```

2. **Hoppscotch**

- https://hoppscotch.io → WebSocket 탭 선택
- URI 입력: `wss://dev.mumulbo.com/ws/chat?token=...`

3. **WebSocket King Client (Chrome 확장)**

- GUI 환경에서 WebSocket 쉽게 테스트 가능

---

## 9. 참고 이미지 및 문서

- 시스템 구조도 (2종)
    - 실시간 채팅 흐름 , 초기 메시지 조회 흐름
        - ![0501_msshim_chat_ar.png](..%2F..%2F9_images%2F0501_msshim_chat_ar.png)
- 채팅 UI 화면정의서 (초기 프로토타입 기준)
- ![0501_msshim_view.png](..%2F..%2F9_images%2F0501_msshim_view.png)

---
