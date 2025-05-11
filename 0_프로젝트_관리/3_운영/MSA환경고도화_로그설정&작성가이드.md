## ⚙️ 로그 설정 가이드

### 1. Docker 로그 드라이버 설정

로그는 `json-file` 드라이버를 사용해 파일 기반으로 수집되도록 설정합니다.

### Project Root/docker-compose.yml 설정

```yaml
logging:
  driver: json-file
  options:
    max-size: "10m"
    max-file: "5"
```

- `max-size`: 로그 파일의 최대 크기 (초과 시 롤링)
- `max-file`: 최대 파일 개수 (순환)

### 2. container_name 설정

컨테이너 이름을 사람이 식별 가능한 형태로 명시합니다.

```yaml
services:
  app:
    container_name: mmb-question-service-dev  # 👈 사람이 알아볼 수 있게 지정
    image: mumulbo/mmb-question-service:dev
    ports:
      - "8081:8080"
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
```

- 이 이름은 **로그 파일 경로에는 반영되지 않지만**,
  `docker logs`, `docker inspect`, `docker exec` 등에서 **사람이 식별하기 쉽게** 도와줍니다.
- 명시하지 않으면 Docker가 무작위 이름(`focused_turing`, `sleepy_morse` 등)을 자동 생성합니다.
- 예: `mmb-ms-msa-playground-app-1` 같은 이름이 `docker compose`에서 자동 생성된 이름입니다.

### 3. 컨테이너 로그 확인 방법

#### 📌 핵심 요약

1. `json-file` 드라이버를 사용할 경우, **로그 경로는 컨테이너 ID 기준으로만 생성되며 변경할 수 없습니다.**
2. 사람이 식별 가능한 로그를 추적하려면 `container_name`을 반드시 지정해야 합니다.
3. 로그 경로를 사람이 보기 쉽게 찾으려면 `docker inspect` 명령을 사용하는 `.sh` 스크립트를 활용하세요.

#### 🔍 로그 경로 예시

```bash
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

#### 🛠 실시간 로그 확인

```bash
sudo tail -f /var/lib/docker/containers/<container-id>/<container-id>-json.log
```

#### 🧪 container_name 으로 로그 경로 찾는 방법

> `.sh` 스크립트를 활용하여 container name 기반으로 로그 경로를 확인할 수 있습니다. 스크립트 예시는 별도 스크린샷으로 제공합니다.

![w9_ms_msaplus_logpath_view.png](..%2F..%2F9_images%2Fw9_ms_msaplus_logpath_view.png)
---

## ✅ 로그 작성 가이드

### 1. 로그 레벨(Level) 설명 및 사용 기준

| 레벨      | 설명                                       | 사용 예시                       |
|---------|------------------------------------------|-----------------------------|
| `INFO`  | 일반적인 상태, 처리 결과, 시작/종료 로그 등 운영환경에서 필요한 정보 | 서비스 시작/종료, 주요 처리 성공 로그      |
| `DEBUG` | 디버깅을 위한 상세 정보. 개발환경에서만 출력                | 요청 파라미터, 내부 흐름 추적           |
| `WARN`  | 문제는 아니지만 주의해야 할 상황                       | 외부 API 응답 지연, 실패 가능성이 있는 상황 |
| `ERROR` | 예외 발생 등 심각한 오류 상황                        | 예외 발생, 처리 실패                |

> ✅ 운영 환경에서는 `INFO` 이상만 출력되도록 설정합니다.

---

### 2. 로그 레벨별 실전 예시

```java
log.info("질문 생성 완료 - questionId={}, userId={}",questionId,userId);
    log.debug("질문 생성 요청 수신 - title={}, tags={}, userId={}",title,tags,userId);
    log.warn("이미 삭제된 질문에 접근 시도 - questionId={}, userId={}",questionId,userId);
    log.error("DB 저장 실패 - userId={}, question={}, error={}",userId,question,e.message,e);
```

---

### 3. 로그 포맷 가이드 - Structured Logging

- 로그는 key=value 구조로 작성해야 함
    - 사람이 보기엔 그냥 문자열 같지만,
      패턴을 정해두면 도구가 key-value처럼 인식할 수 있으므로
      항상 key=value 형식을 유지하는 게 Structured Logging의 핵심입니다.
    - 로그 메시지 패턴이 일관되고 정형적이어서 도구가 정규식 기반으로 쉽게 필드 추출할 수 있음 추출한 필드는 필터링, 집계, 알림 조건으로 사용 가능
    - 실제 예시 (Loki에서 LogQL 쿼리)
        - {app="question-service"} |= "questionId=" | json | questionId="123"
        - grok <log> "%{GREEDYDATA}questionId=%{NUMBER:questionId}, userId=%{NUMBER:userId}"
- 문자열 연결(+)은 비권장
- 성능과 가독성, 파싱 가능성을 고려하여 `{}` 방식 사용

✅ 예:

```java
log.info("질문 저장 완료 - questionId={}, userId={}",questionId,userId);
```

❌ 예:

```java
log.info("질문 저장 완료 - "+questionId+", "+userId);
```

> 기억하기: Structured Logging은 “기계가 이해할 수 있는 로그”다.

---

### 4. 로그 작성 전 고려사항

#### 🔹 생략해도 되는 로그

- 통신 없이 내부 계산만 하는 로직
- 유효성만 통과하는 간단한 흐름
- 초당 수백 번 호출되는 루틴

#### 🔹 포함되면 안 되는 정보

- 전화번호, 주민번호, 토큰, 비밀번호
- 객체 전체 출력 (`toString()` 등)

#### 🔹 자문할 질문

> “이 로그는 운영자가 봤을 때 도움이 되는가?”

> 🧠 DEBUG는 "과정"을, INFO는 "결과"를 남긴다.

---

### 5. 계층별 로깅 기준 (Controller → Service → Repository)

| 계층         | 로그 수준                 | 목적                |
|------------|-----------------------|-------------------|
| Controller | INFO                  | 외부 요청 수신 기록       |
| Service    | DEBUG/INFO/WARN/ERROR | 핵심 비즈니스 흐름        |
| Repository | DEBUG (선택적)           | 조건 분기 또는 복잡 쿼리 추적 

---

### 6. 개선 전/후 예시 (MSA 간 통신)

#### ❌ BEFORE

```java
    MemberResponse member=memberClient.getMember(memberId);
    if(member==null){
    log.error("회원 조회 실패");
    throw new RuntimeException("회원이 존재하지 않음");
    }
```

#### ✅ AFTER

```java
    log.info("🔵 [MemberClient] 회원 정보 요청 - memberId={}",memberId);

    try{
    MemberResponse member=memberClient.getMember(memberId);
    log.info("🟢 [MemberClient] 회원 정보 수신 성공 - memberId={}, nickname={}",memberId,member.getNickname());
    return member;
    }catch(FeignException e){
    log.warn("🟡 [MemberClient] 회원 정보 요청 실패 - memberId={}, status={}, message={}",memberId,e.status(),e.getMessage());
    throw new RuntimeException("회원 조회 실패",e);
    }
```

---

### 7. 로그 메시지 포맷 및 레벨 정책 요약

#### 포맷

```
[Service-Class] 메시지 - key=value
```

#### 레벨 기준

| 레벨    | 의미             |
|-------|----------------|
| DEBUG | 흐름, 파라미터 추적    |
| INFO  | 기능 단위 결과       |
| WARN  | 예외는 아니지만 주의 필요 |
| ERROR | 시스템 오류, 예외 발생  |

---

### 8. 보안 및 성능 체크리스트

#### 🔐 보안

- 절대 출력하지 말 것: 전화번호, 이메일, 토큰, 패스워드
- Member/DTO 객체 자체 출력 ❌

#### ⚠️ 성능

- 문자열 연결 X → `{}` 포맷 사용
- 반복문 안에서 과도한 로그 ❌

---

### 9. 샘플 로그 흐름 시나리오 (MSA 기준)

#### 🔗 요청 시나리오: 클라이언트가 질문 상세 보기 요청

- 사용자 → Gateway → QuestionService → MemberService
- 공통 필드 유지: `requestId=abc123`, `userId=42`

```text
🔵 [Gateway] 요청 수신 - path=/questions/5, requestId=abc123, userId=42
🔵 [Gateway] 라우팅 시작 - target=question-service, requestId=abc123

🟢 [QuestionService] 질문 조회 요청 - questionId=5, requestId=abc123
🟢 [QuestionService] 질문 데이터 조회 성공 - title="로그 구조란?", requestId=abc123

🔵 [MemberClient] 작성자 정보 요청 - memberId=17, requestId=abc123
🟢 [MemberClient] 작성자 정보 수신 성공 - nickname="sinms", memberId=17, requestId=abc123

🟢 [QuestionService] 질문 상세 응답 완료 - questionId=5, responseTime=32ms, requestId=abc123
```

---

### 10. 로거 선언 방식 (Java vs Kotlin)

#### Java

```java
private static final Logger log=LoggerFactory.getLogger(MyService.class);
```

#### Kotlin (확장 함수)

```kotlin
fun <T : Any> T.logger(): Logger = LoggerFactory.getLogger(this::class.java)

class MyService {
    private val log = logger()
}
```

> Java는 확장 함수 문법이 없어 Kotlin처럼 logger() 함수를 사용 불가
