# MSA의 장애 대응 - Part 2. Retry (재시도)

## 1) 🔄 Retry란?
Retry는 **일시적인 실패(transient failure)** 발생 시, 요청을 **자동으로 재시도**하여 복구를 시도하는 장애 대응 전략이다.

예를 들어,
- 네트워크 불안정
- 외부 API 서버 일시 다운
- 데이터베이스 일시적인 커넥션 실패

이러한 상황은 일정 시간 후 재요청 시 성공할 수 있기 때문에, Retry는 복원력(Resilience)을 높이는 데 효과적이다.

---

## 2) ⚙️ Resilience4j의 Retry

Resilience4j는 `@Retry` 애너테이션 또는 `RetryRegistry`(중앙 레지스트리 객체)를 통해 Retry 기능을 제공한다.  
주요 특징은 다음과 같다:

- 실패한 요청을 설정된 횟수만큼 재시도
- 재시도 간 시간 간격 설정 가능
- 재시도 시 로깅 및 fallback 지정 가능
- 특정 예외만 재시도 대상으로 설정 가능

---

## 3) 🔧 설정 방법

### 3.1) 의존성 추가 (build.gradle)
```groovy
implementation("io.github.resilience4j:resilience4j-retry")
```
### 3.2) 설정 예시(application.yml)
```
resilience4j:
  retry:
    instances:
      myService:
        maxAttempts: 3              # 최대 재시도 횟수 (최초 호출 포함)
        waitDuration: 1s           # 재시도 간 대기 시간
        retryExceptions:
          - java.io.IOException    # 재시도 대상 예외
        ignoreExceptions:
          - com.example.MyFatalException  # 재시도 제외 예외
```

## 4) 💡 Retry 동작 흐름 예시
- 설정: `maxAttempts`=3, `waitDuration`=1s
```
[1차 요청 실패]
   ↓ 1초 대기
[2차 요청 실패]
   ↓ 1초 대기
[3차 요청 성공 or 실패 후 fallback]
```

## 5) 🧪 코드 예시
Resilience4j의 Retry는 동기(synchronous) 방식과 비동기(asynchronous) 방식 모두 지원한다.
- 동기 방식
  - 호출이 완료될 때까지 현재 스레드가 기다림
  - 사용되는 경우
    - 즉각적인 응답이 필요한 서비스
    - 내부 HTTP 통신
    - 성능/응답 시간 제어가 중요한 경우(동시 요청 수를 제한하고 싶을 때)
- 비동기 방식
  - 별도 스레드에서 실행되며, CompletableFuture로 결과를 비동기적으로 처리
  - 사용되는 경우
    - 병렬 작업 필요한 경우
    - 외부 시스템 호출 시 I/O 대기시간이 긴 경우
    - 서버 리소스를 절약하고 싶은 경우(블로킹을 줄이면 스레드 수 감소 가능)
    - 서비스 간 호출이 오래 걸리거나 응답을 기다리는 동안 UI나 다른 처리를 계속 진행해야 하는 경우에 특히 유용

### 5.1) 동기 방식(@Retry 어노테이션)
```
@Retry(name = "myService", fallbackMethod = "fallback")
public String callExternalApi() {
    return restTemplate.getForObject("http://external-service/api", String.class);
}

public String fallback(Throwable t) {
    return "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해주세요.";
}
```
- 호출자는 응답을 기다렸다가 받음
- 실패 시 즉시 fallback 실행
- 예) 로그인, 결제 요청 등

### 5.2) 비동기 방식(@Retry + @Async)
```
@Retry(name = "myAsyncService", fallbackMethod = "fallbackAsync")
@Async
public CompletableFuture<String> asyncCall() {
    return CompletableFuture.supplyAsync(() -> {
        String response = restTemplate.getForObject("http://external-service/api", String.class);
        if (response.contains("error")) {
            throw new RuntimeException("API 응답 오류");
        }
        return response;
    });
}

public CompletableFuture<String> fallbackAsync(Throwable t) {
    return CompletableFuture.completedFuture("비동기 처리 실패 시 응답");
}
```
- 호출자는 응답을 기다리지 않고 바로 반환받음
- 실패 시 fallback 도 비동기로 처리
- 최종 결과 (성공 or fallback)가 CompletableFuture나 Mono 등에 담겨 비동기적으로 호출자에게 전달됨
- 예) 알림 전송, 로그 동기화 등

## 6) 📌 Retry 사용 시 주의사항
- 과도한 재시도는 오히려 시스템 부하를 증가시킬 수 있다.
- Idempotent(멱등성) 보장되는 API에만 사용하는 것이 안전하다.
- Retry + TimeLimiter 조합으로 무한 대기를 방지할 수 있다.
- 외부 호출 시에는 반드시 Fallback 처리까지 고려할 것

## 7) ✅ 언제 Retry를 사용할까?
- 네트워크 오류가 간헐적으로 발생하는 경우
- 외부 API가 일시적으로 불안정할 때
- DB 커넥션 오류 등 일시 장애 발생 가능성이 있는 경우
- 단, 사용자 입력 처리나 중복 요청 우려가 있는 경우는 피해야 함