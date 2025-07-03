# MSA의 장애 대응 - Part 1. Resilience4j와 Bulkhead
## 1) 🖼️ 배경
마이크로서비스 아키텍처(MSA)는 각 서비스를 작고 독립적으로 나누어 개발하고 운영할 수 있도록 하여 개발 속도와 확장성을 높이는 장점이 있다. 그러나 시스템이 분산될수록 개별 서비스의 장애가 전체 시스템에 영향을 줄 위험도 증가한다. 

따라서 MSA 환경에서의 장애 대응은 단순히 에러를 처리하는 차원을 넘어, **장애가 발생하더라도 전체 시스템이 살아남을 수 있도록 설계**하는 것이 핵심이다.

이를 가능하게 하는 접근 방식이 바로 **Resilient Architecture (탄력적 아키텍처)** 이며, 이를 지원하는 대표적인 Java 기반 도구가 Resilience4j이다.

## 2) ⚙️ Resilience4j
Resilience4j는 Java 기반 애플리케이션에서 장애 회복과 서비스 안정성을 위한 패턴을 지원하는 라이브러리로, 다양한 resilience 패턴을 통해 장애 상황에 대비할 수 있다.

Netflix Hystrix로 부터 영감을 받아 개발된 Fault Tolerance Library 로, 현재 Netflix Hystrix가 deprecated 되어서, Resilience4j를 권장함.

### 2.1) Resilience4j 주요 기능
---------------
| 기능 이름 | 설명 |
---------|------
Circuit Breaker | 장애가 일정 수준 이상 발생하면 회로를 차단하여 빠르게 실패하게 함
Retry | 실패한 요청을 자동으로 재시도
Rate Limiter | 초당 최대 요청 수 제한 (트래픽 조절)
Bulkhead | 리소스를 격리하여 일부 서비스 장애가 전체에 영향 주지 않도록 방지
Time Limiter | 응답 지연 제한 (타임아웃)
Cache | 동일 요청의 결과를 캐시하여 처리 성능 향상
Fallback | 장애 발생 시 대체 응답을 제공 (Retry 실패나 Circuit Open 시)
-------

### 2.2) Resilience4j 설정
- **의존성 추가**
  - build.gradle에 dependency 추가
    ```
    implementation("org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j:2.1.6")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    ```

- **application 파일 설정**
  - application.yml 또는 application.properties
  - 사용할 인스턴스 이름 별로 설정 (CircuitBreaker 등)
    ```
    resilience4j:
        circuitbreaker:
            instances:
            myService:
                registerHealthIndicator: true
                slidingWindowSize: 10
                minimumNumberOfCalls: 5
                failureRateThreshold: 50 
                waitDurationInOpenState: 5s
                permittedNumberOfCallsInHalfOpenState: 3
                automaticTransitionFromOpenToHalfOpenEnabled: true
        timelimiter:
            instances:
            myService:
                timeoutDuration: 2s
        retry:
            instances:
            myService:
                maxAttempts: 3   # 재시도 횟수(최초 호출 포함)
                waitDuration: 1s # 재시도 사이 간격
        bulkhead:
            instances:
            myService:
                maxConcurrentCalls: 5
                maxWaitDuration: 0
    ```

## 3) ⚙️ Resilience4j - Bulkhead(격리)
Resilience4j의 Bulkhead는 MSA에서 하나의 장애가 전체 시스템으로 전파되지 않도록 격리하는 전략 중 하나이다. 

한 서비스나 리소스가 폭주하더라도 다른 기능은 정상 작동하도록 막아주는 보호막

### 3.1) Resilience4j의 Bulkhead 종류

Resilience4j는 두 가지 유형의 Bulkhead를 제공한다. 

---------------
유형 | 설명
---|---
ThreadPoolBulkhead | 비동기 처리에 적합. <br>고립된 스레드 풀에서 실행함. CompletableFuture와 함께 사용
SemaphoreBulkhead | 동기 처리에 적합. <br> 요청 수를 동시 처리 제한 개수로 제한
---------------

→ 기본적으로 일반 HTTP 요청에는 **SemaphoreBulkhead**를 사용한다.

### 3.2) Bulkhead 주요 설정 값
- application.yml
    ```
    resilience4j:
        bulkhead:
            instances:
            myService:
                maxConcurrentCalls: 5   # 동시에 실행 가능한 최대 요청 수
                maxWaitDuration: 0     # (선택) 대기 가능 시간, 기본값은 0 (즉시 실패)
    ```


---------------
설정 항목 | 설명
---|---
maxConcurrentCalls | 동시에 수행 가능한 호출 수 (초과 시 실패)
maxWaitDuration | (선택) 초과 시 대기할 수 있는 최대 시간. 기본 0 (즉시 거절)
---------------

### 3.3) Bulkhead 사용 예시 (SemaphoreBulkhead)

```
@Bulkhead(name = "myService", type = Bulkhead.Type.SEMAPHORE, fallbackMethod = "fallback")
public String callRemoteService() {
    return restTemplate.getForObject("http://remote-service/api", String.class);
}

public String fallback(Throwable t) {
    return "현재 요청이 많아 응답할 수 없습니다. 잠시 후 다시 시도해주세요.";
}
```

### 3.4) Bulkhead 동작 흐름
3.2, 3.3과 같이 설정한 경우, 
- maxConcurrentCalls = 5이면 동시에 5개 요청만 수행
- 6번째부터는 실패하거나 대기 (설정에 따라 다름)
```
        +-------------------------+
        |  요청 1 (진입)         |
        +-------------------------+
        |  요청 2 (진입)         |
        +-------------------------+
        |  요청 3 (진입)         |
        +-------------------------+
        |  요청 4 (진입)         |
        +-------------------------+
        |  요청 5 (진입)         |
        +-------------------------+
        |  요청 6 (거절 or 대기)|
        +-------------------------+
```

### 3.5) Bulkhead 는 언제 사용될까 ?

- DB 연결 수 제한 필요한 경우
- 외부 API 호출이 느리거나 자주 실패하는 경우
- 트래픽 급증 시, 핵심 기능만 우선 보장하고 싶을 때