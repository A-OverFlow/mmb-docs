# **동시성 이슈(Concurrency Issue)**

동시성은 **여러 작업이 시간적으로 겹쳐서 실행되는 현상**을 뜻합니다.
멀티스레드·멀티프로세스·코루틴·이벤트 루프 등 구현 방식은 다양하지만, 서로 겹쳐 실행되는 작업들이 ‘공유 자원’(메모리·파일·DB 등)에 접근할 때 예상 하지 못한 결과가 발생할 수 있습니다. 이를 총칭해 동시성
이슈라 부릅니다.

# 실무에서 자주 발생하는 동시성 이슈 순위

1. Idempotency(멱등성) 결여로 인한 중복 실행 🔥
    * **개념**
        * 동일 요청이 두 번 이상 도착했을 때 결과가 한 번 처리한 것과 완전히 같아야 한다는 성질이 보장되지 않아, 데이터가 중복 반영되는 문제입니다.
    * **예시**
        * 모바일 앱에서 결제 버튼을 두 번 탭 → 결제 승인 두 건 생성
        * HTTP 502 후 클라이언트가 자동 재시도 → 주문 API가 두 번 호출
        * Kafka 오프셋 커밋이 지연돼 같은 메시지가 다시 소비 → 재고 두 번 차감
2. Lost Update / Race Condition 🔥
    * **개념**
        * 두 개 이상의 트랜잭션이 같은 데이터(특히 같은 행)를 거의 동시에 수정하면서, 마지막에 커밋된 값이 앞선 변경을 덮어써 버리는 현상입니다.
    * **예시**
        * 플래시 세일 중 두 사용자가 동시에 재고-1 요청 → 최종 재고가 1만 감소해야 하지만 2 감소
        * 포인트 적립 로직이 멀티 스레드로 동시에 실행 → 일부 적립 내역이 사라짐
3. 데이터베이스 Deadlock
    * **개념**
        * 트랜잭션 A와 B가 서로 상대가 보유한 락을 기다리며 영원히 풀리지 않는 상태입니다.
    * **예시**
        * 트랜잭션 A: 주문 테이블 → 적립금 테이블 순으로 업데이트
        * 트랜잭션 B: 적립금 테이블 → 주문 테이블 순으로 업데이트
          → 두 트랜잭션이 서로를 기다리며 교착
4. Thundering Herd / Cache Stampede
    * **개념**
        * 캐시 TTL이 동시에 만료되면서 많은 인스턴스가 한꺼번에 원본 데이터베이스나 외부 API를 호출해 폭주가 일어나는 현상입니다.
    * **예시**
        * 자정 0시에 전 제품 가격 캐시가 만료 → 수천 개 애플리케이션 인스턴스가 동시에 DB 조회
        * 인기 게시물 목록 캐시가 10초마다 만료 → 트래픽 피크 시 DB CPU 100 %
5. 분산 정합성(Exactly-Once, Saga·보상 트랜잭션)
    * **개념**
        * 여러 마이크로서비스가 각자 데이터베이스를 가질 때, 단일 사업 행위를 여러 단계로 나누어도 최종적으로 일관된 상태를 유지해야 한다는 과제입니다.
    * **예시**
        * 결제 서비스는 성공·주문 서비스는 실패 → 고객에게 돈은 빠져나갔지만 주문이 없음
        * 배송 상태 이벤트가 중복 전파 → 사용자 화면에 배송 단계가 뒤죽박죽
6. 격리 수준 관련 이상(Dirty/Non-repeatable/Phantom Read)
    * **개념**
        * 낮은 트랜잭션 격리 수준에서 발생하는 읽기 불일치 현상으로, 읽은 데이터가 다른 트랜잭션에 의해 변경·삽입되어 동일 쿼리라도 결과가 달라지는 문제입니다.
    * **예시**
        * 보고서 생성 중 합계가 단계마다 달라져 최종 합계와 상세 수치가 불일치
        * 통계 배치 작업이 진행되는 동안 실시간 API가 중간 상태 데이터를 노출
7. Starvation(기아)·Livelock(라이브락)
    * **개념**
        * 특정 스레드·트랜잭션이 계속해서 자원을 얻지 못하거나, 락을 얻었다 놓았다만 반복해 실제 작업이 진행되지 않는 상태입니다.
    * **예시**
        * CPU-바운드 배치 작업이 스레드 풀 대부분을 점유 → 실시간 API 응답 지연
        * 공정하지 않은 락 구현으로 특정 스레드가 영원히 대기

# Idempotency(멱등성) 보장

## Idempotency 보장 방법 정리

1. **클라이언트(프론트 엔드) 레벨** 🔥
    * **한 번 클릭 후 버튼 비활성화·로딩 스피너 표시**
    * 요청을 보낼 때마다 **UUID·ULID 등으로 Idempotency-Key를 생성**하고, 동일 행위 재시도 시에는 같은 키를 재사용(로컬 스토리지·세션 스토리지 활용)
2. **Idempotency-Key 필터** 🔥
    * `Idempotency-Key` 헤더가 없으면 400 Bad Request
    * Redis `SETNX(Set Not eXists)`로 **“첫 요청만 통과”**
    * 성공 응답을 짧은 TTL(5 – 15 분)로 캐싱 후 **같은 키 재요청에는 그대로 반환**
3. **도메인 모델 · 데이터베이스** 🔥
    * `request_id`(또는 `idempotency_key`) 컬럼에 **UNIQUE 제약** + UPSERT(**UP**date + in**SERT**)
    * JPA `@Version`·CAS(Compare-And-Set)로 Lost Update 방지(중복 write 자체 예방)
4. **메시지 큐 · 이벤트 흐름**
    * **Kafka Transactional Producer** ( `enable.idempotence=true` )
    * **Outbox → CDC(Change Data Capture)** 패턴: 이벤트 저장과 MQ 전송을 원자적으로 묶고, 컨슈머는 `event_id` 중복 검사
5. **관측 · 운영**
    * `idempotency_hit_total`, `duplicate_ratio`, Kafka `record_lag` 등 지표 수집
    * 키 TTL 분포·메모리 사용량 모니터링, 중복 히트 급증 시 UI·네트워크·서버 오류 교차 점검

## Idempotency-Key 필터

### 프런트엔드(React) 측 작업

| 단계               | 핵심 개념 및 목적                                                                                                       |
|------------------|------------------------------------------------------------------------------------------------------------------|
| **행위 ID 생성**     | 글쓰기 페이지(Compose) 진입 순간 **UUID·ULID** 등으로 *행위 고유 ID*를 한 번 생성한다.<br>→ 같은 글 작성을 다시 시도할 때 동일 ID를 재사용하기 위함.           |
| **ID 보관**        | `localStorage` · `sessionStorage` 또는 전역 상태(예: React Context)에 행위 ID를 저장한다.<br>→ 새로고침·네트워크 오류 재시도에도 같은 ID가 유지되도록. |
| **헤더 삽입**        | 모든 제출 요청에 **`Idempotency-Key: {행위 ID}`** 헤더를 포함한다.                                                               |
| **사용자 중복 클릭 방지** | 버튼을 **1회 클릭 직후 즉시 비활성화**하고 로딩 스피너를 보여준다.<br>→ UI 차원에서 중복 요청을 최소화.                                                |
| **실패·재시도 처리**    | 네트워크 오류·5xx 응답 시 **동일 ID로 재전송 가능**하게 버튼을 다시 활성화한다.                                                               |
| **성공 시 정리**      | 글 쓰기가 성공하거나 사용자가 취소·이탈하면 **행위 ID를 storage에서 삭제**해 다음 글 작성 시 새 ID가 생성되도록 한다.                                      |
| **중복 응답 수용**     | 서버가 “이미 처리됨” 응답(같은 200 OK)에 **`duplicate=true`** 등의 플래그를 내려주면,<br>UI 흐름(글 상세 페이지 이동 등)을 정상 성공과 동일하게 처리한다.        |

### 백엔드 측 작업

#### API Gateway(선택적 1차 방어)

| 항목              | 핵심 개념                                                           |
|-----------------|-----------------------------------------------------------------|
| **헤더 필수 검증**    | `Idempotency-Key` 부재 시 400 Bad Request 반환.                      |
| **원자적 첫 요청 판별** | Redis `SETNX`로 **“첫 요청만 통과”**.<br>– TTL 약 3-5 분: 사용자 재시도 허용 범위. |
| **중복 응답 처리**    | 동일 키 재요청은 **즉시 200 OK** + 간단 JSON(`duplicate: true`) 반환.        |
| **키 전달**        | 첫 요청 통과 시 **동일 헤더**를 백엔드 서비스에 그대로 전달한다.                         |
| **지표 수집**       | `gw.idem.duplicate.total`, 중복률 등을 Micrometer → Prometheus 로 노출. |

#### 게시글 서비스(2차‧도메인 방어)

| 항목             | 핵심 개념                                                                               |
|----------------|-------------------------------------------------------------------------------------|
| **키 재검증**      | 서비스 단에서도 `Idempotency-Key`가 존재해야 함을 보장(보안·일관성).                                     |
| **도메인 중복 차단**  | `idempotency_key` 컬럼에 **UNIQUE 제약** + UPSERT OR IGNORE.<br>→ 같은 글이 DB에 이중 삽입되지 않도록. |
| **응답 캐싱(선택)**  | 첫 성공 응답(글 ID 등)을 Redis에 짧게(10-15 분) 저장하여 중복 요청에 재사용.                                |
| **실패 시 롤백 허용** | 트랜잭션 실패·예외 발생 시 키를 삭제하여 정상 재시도를 허용한다.                                               |
| **내부 이벤트 연계**  | 글 작성 이벤트를 Kafka 등으로 발행한다면 **event\_id**에도 행위 ID 또는 UUID를 넣어 컨슈머 중복 처리 방지.           |
| **모니터링**       | `post.idem.dup.hit`, `duplicate_ratio`, DB 락 충돌 지표 등을 대시보드화.                        |

# Lost Update / Race Condition

- Lost Update (손실 업데이트)
    - 두 개 이상의 트랜잭션이 같은 레코드를 읽은 뒤 순차적으로 덮어쓰면서 이전 트랜잭션의 수정 내용을 잃어버리는 현상입니다.
- Race Condition (경쟁 상태)
    - 일련의 연산 순서가 실시간 경쟁(스레드, 트랜잭션, 프로세스) 때문에 매 실행마다 달라져 불안정한 결과가 생기는 현상입니다. Lost Update는 여러 Race Condition 중 하나의 유형입니다.

## Lost Update가 발생하는 전형적 패턴

```text
T1                       T2
 ── READ balance=100 ──►
                        ── READ balance=100 ──►
 ── UPDATE balance=110 ─►
                        ── UPDATE balance=120 ─►   ← T1의 110이 덮여씀
```

*동일 레코드를 여러 트랜잭션이 거의 동시에 읽고(READ) 나중에 쓴(WRITE) 결과가 먼저 커밋된 값을 덮어써 “조용히” 데이터가 손상되는 현상.*

## 해결 전략별 요약

| 전략                                                       | 적용 범위                                                                              | 장점                            | 주의사항                                           |
|----------------------------------------------------------|------------------------------------------------------------------------------------|-------------------------------|------------------------------------------------|
| **Optimistic Lock (`@Version`)**                         | • 동시 충돌이 **드문** 비즈니스 레코드<br>• 읽기 중심(READ-heavy)                                    | 락 점유 시간 0 초                   | 충돌 시 `OptimisticLockException` → **재시도 로직 필수** |
| **Pessimistic Lock (`PESSIMISTIC_WRITE`, `FOR UPDATE`)** | • 충돌이 **잦은** 레코드<br>• “반드시 한 스레드만” 접근해야 하는 핵심 자원                                   | 데이터 정합성 ↑, 재시도 불필요            | 긴 락 보유 → 지연·데드락 가능                             |
| **원자적 UPDATE / UPSERT**<br>(단일 SQL)                      | • 카운터, 재고처럼 **수치 증감**만 필요<br>• `UPDATE … SET stock=stock-1 WHERE id=? AND stock>0` | DB 단일 명령으로 **Race 전부 차단**     | 영향 행 수(0행)로 실패를 구분 → 실패 시 응답·재시도 처리 필요         |
| **애플리케이션-계층 CAS**<br>(Redis `WATCH/MULTI`·Lua)           | • DB 밖 간단 값(재고 캐시, 포인트 등)                                                          | 네트워크 왕복 최소, **초고속**           | 최종 정합성 보증을 위해 주기적 DB 싱크·모니터링 필요                |
| **분산 락**<br>(Redis Redlock, ZooKeeper, etcd)             | • **다중 인스턴스**가 동일 자원 변경<br>• 배치 선점, 크론 동시 실행 방지                                    | JVM 밖 전역 락                    | 타임아웃·노드 분할(brain split) 대비 필요                  |
| **SERIALIZABLE 격리 수준**                                   | • 트랜잭션 수가 많지 않은 **관리형 작업 배치**                                                      | 애플리케이션 코드 수정 없이 **DB가 충돌 감지** | 가장 강한 격리 → 대량 처리 시 롤백·대기 증가                    |

