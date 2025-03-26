작성기한: 2025-03-27

# Kafka 이해하기
## 1. Kafka는 무엇인가
카프카는 분산 스트리밍 플랫폼으로, 대량의 데이터를 처리하고 실시간으로 전송하는 데 사용됩니다.
<br/><br/>

## 2. Kafka 탄생 배경
<img width="600px" src="https://github.com/user-attachments/assets/f478c553-de3d-4fa7-9a4f-20e5cf301db8" />

위 그림은 Kafka 시스템이 도입되기 전 링크드인의 데이터 처리 시스템을 보여줍니다. 이 구조는 두 가지 문제가 있었습니다.
1. 시스템 복잡도(Complexity)의 증가
    * 통합된 전송 영역이 없어 데이터 흐름을 파악하기 어렵고, 시스템 관리가 어려움
    * 특정 부분에서 장애 발생 시 조치 시간 증가 (연결 되어있는 애플리케이션들을 모두 확인해야 했기 때문)
    * HW 교체 혹은 SW 업그레이드 시 관리 포인트가 늘어나고, 작업시간의 증가(연결되어 있는 애플리케이션에 Side Effect가 없는지 확인 필요)
2. 데이터 파이프라인 관리의 어려움
    * 각 애플리케이션과 데이터 시스템 간 별도의 파이프라인이 존재하고 파이프라인마다 데이터 포맷과 처리 방식이 다름
    * 새로운 파이프라인 확장이 어려워지면서, 확장성 및 유연성이 떨어짐
    * 데이터 불일치 가능성이 존재했기 때문에 신뢰도 감소

이에 따라 링크드인은 Kafka 개발팀을 구성해 시스템을 다음과 같이 개선합니다.

![image](https://github.com/user-attachments/assets/16c9b310-8217-4e1c-bc51-21b0f7dbf705)

위 구조는 다음과 같은 이점을 지녔습니다.
1. 기존 end-to-end 통신 방식에서 벗어나 **통합/중앙화된 전송 영역**을 제공한다.
2. 메세지를 생성하는 Producer와 이를 소비하는 Consumer가 **분리됐다**.
3. **대용량** 메세지 처리와 더불어 **빠른** 처리량을 자랑한다.
4. 확장(Scale-out)이 용이한 구조다.
<br/><br/>

## 3. 주요 키워드 및 설명
![image](https://github.com/user-attachments/assets/0582a451-e424-4a53-aba8-a67df07db24f)

* **이벤트(Event):** Kafka을 통해 프로듀서와 컨슈머가 주고받는 데이터 단위, 메세지
* **토픽(Topic):** 데이터의 주제를 나타내며, 이름으로 분리된 로그입니다. 메시지를 보낼 때는 특정 토픽을 지정합니다.
* **파티션(Partition):** 토픽은 하나 이상의 파티션으로 나누어질 수 있으며, 각 파티션은 순서가 있는 연속된 메시지의 로그입니다. 파티션은 병렬 처리를 지원하고, 데이터의 분산 및 복제를 관리합니다.
* **레코드(Record):** 레코드는 데이터의 기본 단위로 키와 값(key-value pair) 구성입니다.
* **오프셋(Offset):** 특정 파티션 내의 레코드 위치를 식별하는 값입니다.
* **프로듀서(Producer):** 데이터를 토픽에 보내는 역할을 하며, 메시지를 생성하고 특정 토픽으로 보냅니다.
* **컨슈머(Consumer):** 토픽에서 데이터를 읽는 역할을 하며, 특정 토픽의 메시지를 가져와서(poll) 처리합니다. 컨슈머 그룹은 여러 개의 컨슈머 인스턴스를 그룹화하여 특정 토픽의 파티션을 공유하도록 구성합니다. 이를 통해 데이터를 병렬로 처리하고 처리량을 증가시킬 수 있습니다.
* **카프카 커넥터(Connector):** 카프카와 외부 시스템을 연동 시 쉽게 연동 가능하도록 하는 프레임워크로 MySQL, S3 등 다양한 프로토콜과 연동을 지원합니다.
  * **소스커넥터(source connector):** 메시지 발행과 관련 있는 커넥터
  * **싱크커넥터(sink connector):** 메시지 소비와 관련 있는 커넥터
<br/><br/>

## 4. 실습 예제
TBD
* [[Spring Boot] Apache Kafka 실습](https://oppr123.tistory.com/57)
* [Kafka Spring Boot 3으로 간단하게 Producer, Consumer 구현해보기](https://yeo-computerclass.tistory.com/546)

<br/><br/>


## 참고 자료
* [우아한형제들 기술블로그 - 우리 팀은 카프카를 어떻게 사용하고 있을까](https://techblog.woowahan.com/17386/)
* [컨플루언트 아티클 - Putting Apache Kafka To Use: A Practical Guide to Building an Event Streaming Platform (Part 1)](https://www.confluent.io/blog/event-streaming-platform-1/)
* [velog - [서버] Kafka 에 대해서](https://velog.io/@choidongkuen/%EC%84%9C%EB%B2%84-Kafka-%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C)

끝.
