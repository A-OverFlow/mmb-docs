# MSA 필수 개념

## ✅ 이미 한 것

## 🔜 앞으로 할 것

## ⚠️ 미흡

## ⛔ 제외

---

1. 마이크로서비스 기본 개념 및 설계 원칙
    * Bounded Context (DDD): 서비스의 경계를 도출하기 위한 핵심 개념 ✅
    * Single Responsibility Principle: 서비스는 하나의 명확한 책임을 가져야 함 ✅
    * 독립적 배포와 개발: 각 서비스는 개별적으로 배포되고 롤백(?) 가능해야 함 ✅
2. 서비스 간 통신 방식
    * 동기 방식 (REST, gRPC) ✅
    * 비동기 방식 (Kafka, RabbitMQ 등) 🔜
    * Synchronous vs. Asynchronous의 장단점과 Trade-off 🔜
3. API Gateway
    * 인증 ✅, 라우팅 ✅, 속도 제한 ⚠️, 로깅 ⚠️, CORS 제어 ⚠️
    * 대표 도구: Spring Cloud Gateway ✅
4. 인증/인가 (Security)
    * Token 기반 인증 (JWT, OAuth2.0) ✅
    * 각 마이크로서비스에서 어떻게 인증 정보를 공유할 것인지 ✅
    * 인증 서버 분리 여부 및 구현 방식 ✅
5. 서비스 디스커버리
    * Eureka, Consul, Zookeeper 등 ⛔
    * 동적 서비스 위치 확인 (특히 클러스터 환경에서 필수) ⛔
6. 장애 복원력 및 회복 전략
    * Circuit Breaker (예: Resilience4j) 🔜
    * Retry / Timeout / Fallback 설계 🔜
    * Bulkhead Pattern, Rate Limiting 🔜
7. 분산 트랜잭션 처리
    * Saga Pattern, Orchestration vs. Choreography 🔜
    * Eventual Consistency 개념에 익숙해야 함 🔜
8. 로그 및 모니터링
    * 분산 로그 수집 (ELK, Loki 등) ⚠️
    * 분산 추적 (Zipkin, Jaeger, OpenTelemetry) ⚠️
    * Prometheus + Grafana 조합을 통한 모니터링 ⚠️
9. 배포 자동화 및 운영
    * CI/CD 파이프라인 구축 (GitHub Actions, Jenkins, ArgoCD 등) ✅
    * 컨테이너 기반 운영 (Docker ✅, Kubernetes ⛔)
    * Blue-Green, Canary Deployment 전략 ⚠️
10. 데이터 관리 전략
    * 각 마이크로서비스의 독립적인 데이터베이스 소유 ✅
    * 데이터 중복 허용과 데이터 동기화 전략 ⚠️
    * CQRS, Event Sourcing의 개념적 이해 🔜
11. 테스트 전략
    * Contract Test (예: Spring Cloud Contract) 🔜
    * 서비스 간 통신이 많아지면서 통합 테스트보다 계약 기반 테스트가 중요 🔜
    * Consumer-Driven Contracts (CDC) 🔜
12. 팀 협업과 조직 설계
    * Conway's Law 이해 🔜
    * MSA는 기술 뿐 아니라 조직 구조와도 깊은 관련 있음 🔜