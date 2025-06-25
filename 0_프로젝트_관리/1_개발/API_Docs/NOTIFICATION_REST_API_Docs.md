# 무물보 API 문서 - Notification Service (개발용)

**Base URL**: `https://dev.mumulbo.com`

---

## 1. 공통

### 1.1. 에러 코드

| HTTP Status Code | Description    | 예시 응답 Body |
|------------------|----------------|------------|
| 401 UNAUTHORIZED | 인증 실패 또는 토큰 누락 | `{}` 또는 없음 |

---

## 2. Notification Service

### 2.1. API 목록

| Method | URL                                            | 설명                    | 인증 방식                                |
|--------|------------------------------------------------|-----------------------|--------------------------------------|
| GET    | `/api/v1/notification`                         | 알림 목록 조회              | `Authorization: Bearer <JWT>`        |
| PATCH  | `/api/v1/notification/read-all`                | 모든 알림 읽음 처리           | `Authorization: Bearer <JWT>`        |
| PATCH  | `/api/v1/notification/{id}/read`               | 특정 알림 읽음 처리           | `Authorization: Bearer <JWT>`        |
| GET    | `/api/v1/notification/unread-count`            | 안 읽은 알림 개수 조회         | `Authorization: Bearer <JWT>`        |
| POST   | `/api/v1/notification/test-notify`             | 알림 테스트용 Kafka 프로듀스 요청 | `Authorization: Bearer <JWT>`        |
| WSS    | `wss://dev.mumulbo.com/api/v1/ws/notification` | 실시간 알림 WebSocket 연결   | Query String에 `token` 포함 (`?token=`) |

---

### 2.2. 알림 목록 조회

* **URL**: `GET /api/v1/notification`
* **설명**: 로그인한 사용자의 전체 알림 목록을 조회합니다.

#### 응답 예시

```json
{
  "data": [
    {
      "id": "685c0ff6e0a92477edcb540c",
      "eventId": "test-event-126",
      "userId": 4,
      "type": "NEW_ANSWER",
      "message": "새로운 알림이 도착했습니다!",
      "relatedUrl": "/questions/1",
      "metadata": null,
      "isRead": false,
      "createdAt": "2025-06-25T15:04:22.776"
    }
  ],
  "success": true,
  "message": null
}
```

---

### 2.3. 모든 알림 읽음 처리

* **URL**: `PATCH /api/v1/notification/read-all`
* **설명**: 로그인한 사용자의 모든 알림을 읽음 처리합니다.

#### 응답 예시

```json
{
  "data": {},
  "success": true,
  "message": null
}
```

---

### 2.4. 특정 알림 읽음 처리

* **URL**: `PATCH /api/v1/notification/{id}/read`
* **설명**: 지정한 알림 ID를 읽음 처리합니다.

#### 요청 예시

```
PATCH /api/v1/notification/685c0ff6e0a92477edcb540c/read
```

#### 응답 예시

```json
{
  "data": {},
  "success": true,
  "message": null
}
```

---

### 2.5. 안 읽은 알림 개수 조회

* **URL**: `GET /api/v1/notification/unread-count`
* **설명**: 현재 로그인한 사용자의 읽지 않은 알림 개수를 반환합니다.

#### 응답 예시

```json
{
  "data": {
    "unreadCount": 1
  },
  "success": true,
  "message": null
}
```

---

### 2.6. 알림 테스트 메시지 전송

* **URL**: `POST /api/v1/notification/test-notify`
* **설명**: Kafka를 통해 수동으로 알림을 발송합니다 (테스트 용도).

#### 요청 예시

```json
{
  "eventId": "test-event-126",
  "notificationType": "NEW_ANSWER",
  "receiverUserId": 4,
  "message": "새로운 알림이 도착했습니다!",
  "relatedUrl": "/questions/1"
}
```

#### 응답 예시

```json
{
  "data": {},
  "success": true,
  "message": null
}
```

---

### 2.7. 실시간 알림 WebSocket 연결

* **URL**: `wss://dev.mumulbo.com/api/v1/ws/notification?token=<JWT>`
* **설명**: WebSocket을 통해 실시간 알림을 수신합니다.

#### 연결 예시

```
wss://dev.mumulbo.com/api/v1/ws/notification?token=eyJhbGciOi...<생략>
```

#### 응답 메시지 예시

```json
{
  "eventId": "test-event-126",
  "receiverUserId": 4,
  "notificationType": "NEW_ANSWER",
  "message": "새로운 알림이 도착했습니다!",
  "relatedUrl": "/questions/1"
}
```

---

## 3. 다른 서비스에서 Kafka로 알림 전송하기

### 3.1. 전송 대상 Kafka 토픽

```
notification-events
```

### 3.2. 전송 메시지 구조

```json
{
  "eventId": "test-event-126",
  "notificationType": "NEW_ANSWER",
  "receiverUserId": 4,
  "message": "새로운 알림이 도착했습니다!",
  "relatedUrl": "/questions/1"
}
```

### 3.3. Java 예제

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendNotification(NotificationEventDto dto)throws JsonProcessingException{
    ObjectMapper objectMapper=new ObjectMapper();
    String json=objectMapper.writeValueAsString(dto);
    kafkaTemplate.send("notification-events",json);
    }
```

### 3.4. Kotlin 예제

```kotlin
@Autowired
lateinit var kafkaTemplate: KafkaTemplate<String, String>

fun sendNotification(dto: NotificationEventDto) {
    val json = objectMapper.writeValueAsString(dto)
    kafkaTemplate.send("notification-events", json)
}
```

> 💡 `NotificationEventDto`는 알림 구조를 담은 DTO이며, 메시지 필드를 위의 JSON 구조에 맞춰 정의하면 됩니다.
> Kafka 설정이 안 되어 있는 서비스에서는 `spring.kafka.bootstrap-servers` 설정이 필요합니다.

![w17_notification_1.png](..%2F..%2F..%2F9_images%2Fw17_notification_1.png)
![w17_notification_2.png](..%2F..%2F..%2F9_images%2Fw17_notification_2.png)
![w17_notification_3.png](..%2F..%2F..%2F9_images%2Fw17_notification_3.png)
![w17_notification_4.png](..%2F..%2F..%2F9_images%2Fw17_notification_4.png)
![w17_notification_5.png](..%2F..%2F..%2F9_images%2Fw17_notification_5.png)
![w17_notification_6.png](..%2F..%2F..%2F9_images%2Fw17_notification_6.png)
![w17_notification_7.png](..%2F..%2F..%2F9_images%2Fw17_notification_7.png)