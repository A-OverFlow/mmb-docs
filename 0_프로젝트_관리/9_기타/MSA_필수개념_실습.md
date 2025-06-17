# 공통 사항

- **반드시 MMB 프로젝트에 적용하여 실습**
- 문서 작성
    - 개념 정리
    - 실습 과정
    - 가이드

# [조영용] 동시성 문제 해결 방법

## 중복 요청 처리 (Duplicate Request Handling)

- 사용자 또는 시스템이 같은 요청을 여러 번 보낼 수 있음 (예: 결제 API 버튼 중복 클릭)
- 재시도(Timeout 후 Retry)로 인해 서비스가 같은 요청을 중복 처리할 수 있음

## 재고 감소/포인트 차감 동시 요청

- 여러 인스턴스에서 동시에 같은 재고를 차감하려 할 때, 재고가 마이너스가 되는 경우 발생
- 경쟁 조건(Race Condition)으로 데이터 정합성이 깨짐

## 이벤트 중복 소비

- Kafka, RabbitMQ 등에서 이벤트가 중복 전송 또는 재시도되어 같은 이벤트가 여러 번 처리될 수 있음

## 보상 트랜잭션 중 충돌

- SAGA 패턴에서 보상 트랜잭션 실행 중, 동시에 다른 요청이 들어오면 데이터 정합성이 깨짐

## 캐시와 DB 간 동기화 불일치

- 캐시 → DB 순으로 업데이트할 경우 캐시 쓰기 성공 후 DB 쓰기 실패 시 데이터 불일치
- 특히 Redis와 DB를 분리해서 쓸 때 자주 발생

## 동시 API 호출로 인한 상태 중복 처리

- 예: 주문 생성과 동시에 상태 변경 API가 들어와 처리 순서 꼬임
- 예: 동일 사용자가 동시에 여러 창에서 같은 작업을 시도

# [송준희] 분산 추적 & 로그 중앙화

## 왜 분산 추적과 로그 중앙화는 함께 이루어져야 하는가?

- 분산 추적은 흐름(Trace), 로그는 내용(Log)
    - **분산 추적**은 “어느 서비스가 언제 호출되었는가?”를 보여줍니다.
    - **로그 중앙화**는 “해당 시점에 무슨 일이 있었는가?”를 보여줍니다.
- traceId로 로그와 트레이스를 연계해야 원인 파악이 가능
    - traceId가 없다면 로그는 수많은 요청 속에서 특정 요청을 찾기 어렵고,
    - 분산 추적만 있다면 **예외 메시지, 파라미터, 내부 상태**는 확인할 수 없습니다.
- 운영/모니터링 도구도 둘의 통합을 기본 전제로 발전 중
    - Grafana, Elastic, Datadog, New Relic 등 대부분의 플랫폼이 **trace + log + metric의 통합 뷰**를 제공합니다.
    - Spring Boot도 이를 위한 표준화를 **Micrometer + OpenTelemetry**로 통합 중입니다.

## Spring Boot 환경에서 추천되는 기술 조합

| 기능          | 기술 스택                                              | 설명                                                    |
|-------------|----------------------------------------------------|-------------------------------------------------------|
| **분산 추적**   | Micrometer Tracing + OpenTelemetry + Grafana Tempo | Sleuth의 공식 후속 조합. OTLP 기반, 표준화 지원                     |
| **로그 중앙화**  | Logback MDC(traceId 포함) + Loki + Grafana           | 로그에 traceId 자동 삽입 → Grafana Explore에서 trace-log 연동 가능 |
| **모니터링 통합** | Grafana UI                                         | trace + log + metric 통합 시각화                           |
| **수집기**     | OpenTelemetry Collector + Fluent Bit               | 각각 trace, log를 중앙으로 전송                                |

## Zipkin이 추천되지 않는 이유

1. **Spring Cloud Sleuth 폐기됨**
   → Sleuth는 더 이상 유지보수되지 않으며, Zipkin과의 연동이 주로 Sleuth에 의존
2. **표준 미지원**
   → OpenTelemetry(OTLP 등)와의 통합성이 떨어짐
3. **확장성/운영성 낮음**
   → 대규모 운영에 적합하지 않으며, 기능이 제한적
4. **활발한 발전 없음**
   → Jaeger, Tempo 등 CNCF 기반 도구에 비해 커뮤니티나 기능 발전이 느림

# [이윤진, 최유연] 장애 대응

- MSA에서 장애 대응의 목적은 단순한 "에러 처리"가 아니라, 장애가 나도 전체 시스템이 살아있도록 만드는 생존 전략
- 장애는 피할 수 없기 때문에, **"장애를 견디는 설계(Resilient Architecture)"**가 MSA 운영의 핵심

## Rate Limiting (API Gateway에 적용)

## Resilience4j (애플리케이션에 적용)

### 서킷 브레이커

### Retry + Backoff

### TimeLimiter (시간 제한)

### Bulkhead (격리)

### Fallback (대체 응답)

# [심명섭] 서비스 간 데이터 관리

- 조인이 불가능한 상황에서 어떻게 필요한 정보를 가져올 것인가?

## API Composition (API 조합 방식)

- 데이터를 직접 조인하지 않고, 각 서비스의 API를 호출하여 필요한 데이터를 조합
- 일반적으로 API Gateway나 BFF(Backend for Frontend) 계층에서 수행

## Command-Query Responsibility Segregation (CQRS)

- **Command(쓰기)**와 **Query(읽기)**를 분리하여,
- Command는 개별 서비스가 처리
- Query는 데이터를 복제하거나 조합해서 처리 (View DB 사용 가능)

## Event Sourcing

# [공희재] 분산 트랜잭션

- 비즈니스 프로세스를 구성하는 다수의 서비스 호출을 어떻게 하나의 일관된 흐름으로 보장할 것인가?
- 어떤 단계에서 실패했을 때 어떻게 보상할 것인가?

## SAGA 패턴 (분산 트랜잭션 처리)

- 하나의 비즈니스 트랜잭션을 여러 로컬 트랜잭션으로 나눠서 처리
- 실패 시 보상 트랜잭션(rollback-like 동작)을 호출함
- 패턴 종류
    - Orchestration 방식: 중앙 조정자가 전체 트랜잭션 흐름 제어
    - Choreography 방식: 서비스들이 이벤트 기반으로 서로 조율

## Event-Driven Architecture + 비동기 통신

- 데이터 변경을 이벤트로 발행하고, 다른 서비스가 이벤트를 수신해서 처리
- Kafka, RabbitMQ 등을 사용

## Outbox 패턴 + CDC (Change Data Capture)

- 서비스 내 트랜잭션에서 이벤트를 Outbox 테이블에 먼저 저장하고,
- CDC 또는 백그라운드 프로세스가 Kafka로 전송
