# 들어가며...
각 서비스별로 의존성이 있는 경우 (공통모듈서비스와 이에 의존하는 서비스들, 혹은 MSA와 같이 서로의 기능을 호출하는 구조)

당장의 실시간 정보를 가져올 수 있는 동기적인 연동을 하게 된다.

참고) 동기적인 연동과 비동기적인 연동
```
A 서비스 → B 서비스에 HTTP 요청
         ↳ B 서비스가 응답할 때까지 A 서비스는 대기함

```

```
A 서비스 → 메시지 큐(Kafka, RabbitMQ 등)에 요청 데이터 전송
         ↳ B 서비스는 큐에서 메시지를 소비함
         ↳ A는 응답을 기다리지 않고 바로 다음 작업 진행

```
동기적인 연동은 당장의 실시간 정보를 가져올 수 있다는 장점이 있지만, 서비스간에 강력한 의존 관계가 생기고
쉽게 장애 전파가 된다.

만약, 비동기적으로 연동이 된다면 직접적인 장애 전파를 막을 수 있지만,
실시간성 처리나 비동기 처리를 위한 코드 복잡도 상승 등의 이유로 비동기 전환이 어려울 수 있다.

따라서 서킷 브레이커는 동기 상황에서 장애 전파를 위해 필요하다.


# 서킷 브레이커란?
누전차단기, 회로차단기, 주식 시장에서 흔히 볼 수 있는 용어로 

과열된 회로를 차단하거나 주가 변동으로 인한 시장 붕괴를 막기위한 거래를 중단하는 장치/제도를 일컫는다.

다들 장애 전파를 막는데에 목적을 두고 있다.

SW에서 말하는 서킷 브레이커는 서로 다른 시스템 간의 연동 시, 장애 전파 차단을 목적으로 한다.   
연동 시 이상을 감지하고, 이상이 발생하면 연동을 차단하고, 이후 이상이 회복하면 자동으로 다시 연동하기 위한 기술이다.

서킷 브레이커는 상태에 따라 서로 다른 동작을 하는데, 

3가지의 보통 상태(OPEN, CLOSED, HALF_OPEN)와 2가지의 특별한 상태(DISABLED, FORCE_OPEN)을 갖는다.   
보통 상태엔
- OPEN/HALF_OPEN : 문제 발생이 감지됨
- CLOSED : 정상적으로 호출되고 응답을 줌

특별한 상태엔
- DISBLED : 항상 호출을 허용
- FORCE_OPEN : 항상 호출을 거부

서킷브레이커는 슬라이딩 윈도(sliding window)를 사용하여 상태의 변화여부를 결정한다.

슬라이딩 윈도는 횟수 방식(COUNT_BASED)과 시간 방식(TIME_BASED)으로 나뉜다.   

###### 슬라이딩 윈도 방식이란,    
###### 최근 일정 범위(Count Based, Time Based) 내의 호출 결과(성공/실패)를 기준으로 실패율을 계산하는 방식을 일컫는다.   
###### 오래된 호출 결과는 무시하고, **지속적으로 최신 호출 기록만 유지** 하면서 상태(Open/Closed)를 판단한다.   



###### 횟수 방식과 시간 방식이란,
###### - 횟수 방식 (count-based) :  최근 N번 호출 중 실패율로 판단 ( ex: 최근 10번 중 5번 실패 -> 50% 실패율)
###### - 시간 방식 (time-based) : 최근 T초 동안의 호출 중 실패율로 판단 ( ex: 최근 10초 동안 5번 중 3번 실패)   



방식에 따라 슬라이딩 윈도 안에서 정해진 확률보다 높은 확률로 호출에 실패하게 되면 상태를 OPEN으로 변경한다.

OPEN 상태에서는 연동된 시스템 호출을 시도하지 않으며, 
바로 호출 실패 Exception을 발생시키거나 정해진 fallback 동작을 수행한다.

OPEN 이후 설정한 시간이 지나면 HALF_OPEN 상태로 변경되며, 
호출이 정상화되었는지 다시한번 실패 확률로 확인한다.

정상화되었다고 판단되면, 
CLOSED 상태로 변경되며, 아직 정상화되지 못했다고 판단되면 다시 OPEN 상태로 되돌아 간다.
<img width="479" alt="image" src="https://github.com/user-attachments/assets/9e113226-5638-4ae1-911a-652046903fde" />

# Resilience4j
애플리케이션 레벨에서 서킷브레이크 구현에는 Resilience4j를 사용한다.

Resilience4j는 Netflix Hystrix로부터 영감을 받은 함수형 프로그래밍으로 설계된, 경량의 내결함성 라이브러리이다.

함수형 프로그래밍으로 설계가 되어서 functional interface, lambda, method reference 등을 활용하여 구현이 가능하다.

###### functional interface, lambda, method reference   
###### functional interface : 메서드가 하나만 선언된 인터페이스 (@FunctionalInterface 사용)   
###### Lambda : 익명 함수를 간단히 표현하는 문법 (param -> logic)   
###### Method Reference : 기존 메서드를 람다처럼 쓰는 문법 (ClassName::methodName)   

원래 Spring Cloud에서는 Spring Cloud Hystrix를 서킷 브레이커로 사용했으나, 
현재는 maintenence 모드로 전환되었고, 대체 모듈로 Resilience4j가 선택되었다.
참고로, Resilience4j는 Rate Limiter, Retry, Bulkhead, TimeLimiter, Cache등을 수행할 수 있다.

## 코어 모듈
```
dependencies {
  // 1. CircuitBreaker : 장애 전파 방지 기능 제공
  implementation("io.github.resilience4j:resilience4j-circuitbreaker:${resilience4jVersion}")
  // 2. Retry : 요청 실패 시 재시도 처리 기능 제공
  implementation("io.github.resilience4j:resilience4j-retry:${resilience4jVersion}")
  // 3. RateLimiter : 제한치를 넘어서 요청을 거부하거나 Queue 생성하여 처리하는 기능 제공
  implementation("io.github.resilience4j:resilience4j-ratelimiter:${resilience4jVersion}")
  // 4. TimeLimiter : 실행 시간제한 설정 기능 제공
  implementation("io.github.resilience4j:resilience4j-timelimiter:${resilience4jVersion}")
  // 5. Bulkhead : 동시 실행 횟수 제한 기능 제공
  implementation("io.github.resilience4j:resilience4j-bulkhead:${resilience4jVersion}")
  // 6. Cache : 결과 캐싱 기능 제공
  implementation("io.github.resilience4j:resilience4j-cache:${resilience4jVersion}")
}
```

## 모듈의 우선순위   
Resilience4j에서 각 모듈은 다음과 같은 데코레이터 구조로 중첩 적용된다.   
Retry ( CircuitBreaker ( RateLimiter ( TimeLimiter ( BulkHead ( TargetFunction ) ) ) ) )

다만, Spring AOP 기반의 실행 우선순위 기준으로는, @Retry의 우선 순위 값이 더 작기 때문에 Retry가 CircuitBreaker보다 먼저 실행된다.   
resilience4j의 CircuitBreakerConfigurationProperties, RetryConfigurationProperties 클래스 내부를 살펴보면,

CircuitBreakerConfigurationProperties
```java
public class CircuitBreakerConfigurationProperties extends
    io.github.resilience4j.common.circuitbreaker.configuration.CircuitBreakerConfigurationProperties {
    private int circuitBreakerAspectOrder = Ordered.LOWEST_PRECEDENCE - 3;
}

```

RetryConfigurationProperties
```java
public class RetryConfigurationProperties extends
    io.github.resilience4j.common.retry.configuration.RetryConfigurationProperties {
    private int retryAspectOrder = Ordered.LOWEST_PRECEDENCE - 4;
}
```

CircuitBreakerAspect
```java
@Aspect
public class CircuitBreakerAspect implements Ordered {
   @Override
    public int getOrder() {
        return circuitBreakerProperties.getCircuitBreakerAspectOrder();
    }
}
```

AOP 기반하에 동작하므로 우선순위를 바꿔서 적용하고자 할 경우, annotation 방식을 사용하여 layer를 분리하거나
aspectOrder 속성 값을 수정하여 적용할 수 있음
###### 서킷브레이커는 AOP(관점지향프로그래밍)로 동작하므로, 다른 AOP보다 먼저/나중에 실행되도록 우선순위를 조정할 수 있다는 뜻입니다.   
###### 방법 1. @Order annotation으로 적용 순서를 직접 지정   
###### 방법 2. AOP 설정 클래스에서 aspectOrder 속성값을 조정하여 실행 우선순위 제어




## 코드 적용


```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import io.github.resilience4j.circuitbreaker.CallNotPermittedException;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class ExternalServiceExecutor {

    // name = "externalService" : 설정 파일에서 이 이름으로 서킷 설정을 찾음
    // fallbackMethod = "fallback" : 호출 실패 시 실행할 대체 메서드 지정
    @CircuitBreaker(name = "externalService", fallbackMethod = "fallback")
    public ServiceResponse callExternalService(ServiceRequest request) {
        // 실제 외부 API 호출 로직
        // ...
        return new ServiceResponse(); // 예시 응답
    }


    // 기본 fallback
    // 일반적인 모든 예외를 처리하는 fallback
    // 외부 시스템에서 타임아웃, 연결 실패, 런타임 예외 등 어떤 예외가 발생하든 이 메서드로 fallback 가능.
    private ServiceResponse fallback(ServiceRequest request, Exception exception) {
        log.warn("Fallback executed due to exception. request: {}", request, exception);
        return getDefaultResponse(request);
    }

    // CircuitBreaker가 열렸을 때만 대응하는 fallback
    // 서킷 브레이커가 Open 상태일때만 호출됨
    // "너무 자주 실패해서 이제 호출 자체를 차단 중이야"라는 상황을 의미
    // 실무에서는 이 fallback에만 특별한 메시지를 넣거나 모니터링 알람을 걸기도 함
    private ServiceResponse fallback(ServiceRequest request, CallNotPermittedException exception) {
        log.warn("[CircuitBreaker: OPEN] Fallback executed. request: {}", request, exception);
        return getDefaultResponse(request);
    }

    // fallback 메서드들에서 공통으로 쓰이는 기본 응답 생성 메서드
    // 실패했지만 사용자에게는 최소한의 안내 메시지를 주기 위한 용도   
    private ServiceResponse getDefaultResponse(ServiceRequest request) {
        // 실패 시 반환할 기본 응답 처리 로직
        return ServiceResponse.failure("Temporary service unavailable");
    }
}

```
   
예외별 fallback 오버로딩을 처리 : Exception vs CallNotPermittedException   
응답 일관성 확보 : 모든 실패 상황에서도 ServiceResponse.failure(...)로 응답 포맷 유지   
비즈니스 로직 분리 : 외부 호출/대응 분리로 구조 명확   
설정 파일 연동 :  externalService 이름으로 yml 에서 설정 가능 (silidingWindowSize, failureRateThreshold 등)   

# 무물보에서 쓰려면...
- @CircuitBreaker 어노테이션을 활용해 실패 감시 + 차단 + fallback 적용
- 예외별로 fallback 메소드를 오버로딩해서 정교하게 제어
- 응답 포맷이나 로깅도 통일성 있게 구성

