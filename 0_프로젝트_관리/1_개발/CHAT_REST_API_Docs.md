# 무물보 API 문서 - Chat Service

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

| HTTP Status Code     | Custom Error Code | Description | 응답 예시(Body)                                                                     |
|----------------------|-------------------|-------------|---------------------------------------------------------------------------------|
| **401** UNAUTHORIZED | `AUTH-4000`       | 인증되지 않음     | <pre lang="json">{&#13;  "code": "AUTH-4000",&#13;  "message": null&#13;}</pre> |
| **401** UNAUTHORIZED | `AUTH-4001`       | 유효하지 않은 토큰  | <pre lang="json">{&#13;  "code": "AUTH-4001",&#13;  "message": null&#13;}</pre> |

<br/><br/>

---

## 2. Chat Service

### 2.0. API 리스트

| HTTP Method | URL                                      | 비고         |
|-------------|------------------------------------------|------------|
| GET         | https://mumulbo.com/api/v1/chat/messages | 채팅 목록 조회   |
| POST        | https://mumulbo.com/api/v1/chat/message  | 채팅 저장      |
| wss         | https://mumulbo.com/api/v1/chat/         | 실시간 채팅 웹소켓 |

<br/>

---