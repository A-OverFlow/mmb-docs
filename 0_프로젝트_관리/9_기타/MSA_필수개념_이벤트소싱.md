# 📌 Event Sourcing 개념공부

## 1. Event Sourcing 개념

- **Event Sourcing**은 데이터를 직접 저장하지 않고, **도메인 이벤트(무슨 일이 일어났는지)**만 순차적으로 저장하는 아키텍처 패턴입니다.
- 현재 시스템의 상태는 저장된 이벤트들을 시간 순서대로 **리플레이(재생)**해서 계산합니다.

### ✅ 핵심 특징
| 항목                | 설명                                                                                   |
|---------------------|------------------------------------------------------------------------------------------|
| **Source of Truth** | 이벤트 자체가 시스템의 진실한 상태 데이터입니다.                                         |
| **리플레이(Replay)**| 이벤트들을 시간순으로 재적용하여 현재 상태를 복원합니다.                                  |
| **확장성**           | 추가 로직 없이 새로운 뷰나 기능을 이벤트만으로 쉽게 구성할 수 있습니다.                |

---

## 2. Event Sourcing vs 단순 로그 기록

| 항목                      | **단순 로그(Log)**                                                | **Event Sourcing**                                                           |
|---------------------------|-------------------------------------------------------------------|-------------------------------------------------------------------------------|
| **목적**                   | 장애 분석, 감사 용도로 단순히 기록                                | 시스템의 **진짜 상태를 구성하는 핵심 데이터**                                 |
| **데이터 위치**            | DB 외부(파일, 로그 시스템 등)에 기록                              | DB에 이벤트 자체가 **“진짜 데이터”로 저장**                                   |
| **데이터 정합성**           | 로그 유실되어도 시스템 상태에는 영향 없음                         | 이벤트 유실 = 시스템 상태 손상 → **트랜잭션처럼 보장 필요**                  |
| **조회 방식**              | DB의 현재 상태를 따로 저장 → 로그는 참고용                        | 이벤트들을 시간순으로 재생(Replay) → 현재 상태 계산                          |
| **역할**                   | 단순 “발생 사실 기록”                                             | 상태 변경의 **유일한 소스(Source of Truth)**                                  |
| **활용**                   | 장애 추적, 모니터링, 감사                                         | 상태 복원, 타임머신(특정 시점 복원), 이벤트 기반 아키텍처 확장               |

---

## 3. 채팅 서비스 기존 구조

```
[WebSocket 메시지 수신]
    ↓
[Redis Pub → RedisSubscriber 수신]
    ↓
[ChatMessage 저장 (MongoDB)]
    ↓
[GET /rooms/{id}/messages → Mongo에서 조회]
```

- 메시지 전송은 Redis Pub/Sub로 처리됨 (비동기)
- MongoDB에 상태(`ChatMessage`)를 직접 저장

---

## 4. Event Sourcing 적용 후 변경 구조

```
[WebSocket 메시지 수신]
    ↓
[Redis Pub → RedisSubscriber 수신]
    ↓
[ChatEvent 저장 (MongoDB)]
    ↓
[GET /rooms/{id}/messages → 이벤트 리플레이 or View 조회]
```

- 상태 대신 **ChatEvent**를 저장
- 조회 시 **이벤트들을 시간 순서대로 리플레이**하여 메시지 목록 생성
- 또는 리플레이 결과를 **ChatMessageView 컬렉션**에 저장해 빠르게 조회

---

## 5. 적용 로직 설계

### ✅ 5.1. ChatEvent 모델 정의

```kotlin
data class ChatEvent(
    val eventType: String, // ex) "MESSAGE_SENT", "MESSAGE_DELETED"
    val roomId: String,
    val senderId: String,
    val content: String?, // 메시지 내용
    val timestamp: Instant,
    val targetMessageId: String? = null // 삭제/수정용 식별자
)
```

---

### ✅ 5.2. 메시지 저장 → 이벤트 저장으로 변경

```kotlin
val event = ChatEvent(
    eventType = "MESSAGE_SENT",
    roomId = roomId,
    senderId = senderId,
    content = message,
    timestamp = now
)
chatEventRepository.save(event)
```

---

### ✅ 5.3. 메시지 조회 시 리플레이 적용

```kotlin
val events = chatEventRepository.findByRoomIdOrderByTimestamp(roomId)
val messages = mutableListOf<ChatMessageView>()

for (event in events) {
    when (event.eventType) {
        "MESSAGE_SENT" -> messages.add(ChatMessageView(...))
        "MESSAGE_DELETED" -> messages.removeIf { it.id == event.targetMessageId }
    }
}
return messages
```

또는, 리플레이 결과를 `ChatMessageView`에 저장해두고 API에서는 View만 조회하는 방식으로 처리

---

## 6. 성능 최적화 방법

| 최적화 기법       | 설명                                                                                 |
|--------------------|----------------------------------------------------------------------------------------|
| **스냅샷(Snapshot)** | 일정 시점의 상태를 저장해두고, 이후 이벤트만 리플레이하여 속도 개선                    |
| **Projection**      | 이벤트를 처리할 때마다 View DB에 상태를 업데이트 → 조회 시 즉시 응답 가능               |
| **리플레이 범위 제한**| 오래된 이벤트는 무시하고 최근 이벤트만 재생 (예: 최근 100개 메시지)                     |

---

## 7. Event Sourcing에 적합한 도메인 예시

| 도메인               | Event Sourcing 미적용 시 문제                                                        | Event Sourcing 적용 시 이점                                                       |
|-----------------------|----------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| 🏦 **은행 계좌**       | 잔액만 저장 → 입출금 내역 추적 불가                                                   | 모든 입출금 이벤트 기록, 과거 특정 시점 잔액 복원 가능                              |
| 📦 **주문/배송 관리**  | 상태 컬럼만 변경 → 상태 전환 과정 유실                                                | `ORDER_PLACED`, `SHIPPED` 등 상태 전환 이벤트 기록, 변경 이력 추적 가능             |
| 🎮 **게임 로그**       | 최종 상태만 저장 → 버그 재현 및 분석 어려움                                           | 모든 플레이어 액션 기록, 과거 게임 상태 재현 가능                                  |
| 📈 **주식 거래**       | 최종 잔고만 저장 → 거래 이력 불투명                                                   | 거래 이벤트 기록, 규제기관 감사 대응 가능                                           |
| 🛒 **쇼핑몰 장바구니** | 최종 장바구니 상태만 저장 → 세션 유실 시 데이터 복원 불가                             | 이벤트 재생으로 장바구니 상태 복원 가능                                             |

---

## ✅ 결론

Event Sourcing 개념을 도입하면 다음과 같은 장점이 생깁니다:

- **모든 메시지 변경 이력 추적 가능**
- **과거 특정 시점 상태 복원 가능**
- **기능 확장성 증가 (메시지 삭제, 수정, 읽음 처리 등)**

---

## 📝 적용 계획 (적용은 못할듯 함 . . .)

| 단계 | 작업 내용                                                   |
|------|--------------------------------------------------------------|
| 1    | `ChatEvent` 클래스 및 저장소 정의                             |
| 2    | 기존 `ChatMessage` 저장 로직 → `ChatEvent` 저장으로 변경      |
| 3    | 리플레이 로직 작성 → 메시지 목록 구성                          |
| 4    | Projection 로직 작성 → View DB에 상태 미리 갱신 (선택)         |
| 5    | API 수정 → `GET /rooms/{id}/messages`에서 View 또는 리플레이 결과 반환 |

---
