심심해서 적어보는 알림서비스 SRS . . .

1. 기능
   - 내 질문에 누가 답변했을 때 (진행)
   - 새로운 질문이 등록되었을때 (너무많으려나?)
   - 신규 회원이 가입했을때 (가입자수가 많지않기때문에 넣어도 될 듯)
   
   아래는 무물보 2.0 쯤?
   - 내 답변에 누가 댓글을 남겼을 때 
   - 질문이 채택되었을 때
   - 내 질문에 누군가 좋아요를 눌렀을 때
   - 내가 팔로우한 사용자가 질문을 올렸을 떄

2. 지원 기능
 - GET /api/v1/notifications (현재 로그인한 사용자의 알림 목록 , 최초로그인시 화면에 보여주기 위함)
 - PATCH /api/v1/notifications/{id}/read (읽음 처리)
 - /api/v1/notifications/unread-count (안 읽은 알림 수 조회)
 - 실시간 알림 전송(WebSocket + Redis Pub/Sub)
 - PATCH /notifications/read-all (전체 읽음 처리)


2. 아키텍쳐

[질문/답변/회원 서비스 중 알림이 필요한 경우 kafka produce]
   → KafkaTemplate.send("notification-events", json)
       ↓
[알림 서비스]
   - Kafka Consumer: 메시지 수신
   - MongoDB 저장: 알림 영속화
   - Redis Pub/Sub: 실시간 브로드캐스트
   - WebSocket: 접속 중인 유저에게 알림 전송

2.0 리소스
  - ec2 에 램이 3g 정도 밖에 남지않았기때문에 리소스 최소화해야함
    - app 500mb
    - redis 100mb
    - kafka + zookeper 500mb + 300mb
    - mongodb 300mb

2.1 Kafka 사용이유
  - Eventual Consistency 개념 실습 
    - 즉시 일관성은 보장하지 않지만, 시간이 지나면 데이터 일관성이 맞춰진다"는 의미
    - 서비스 간 직접 호출 대신, 메시지 큐(Kafka 등)로 비동기 전파하면 각 서비스는 일관성을 조금 늦게 갖게 됨
    - 메시지를 저장, 재처리, 재시도할 수 있는 장치를 통해 궁극적 일관성 확보
    
  -  서비스 간 결합도 최소화
    - 질문/답변 서비스는 알림 서비스를 전혀 몰라도 됨
     
  - 재시도/내구성 보장/대용량처리
    - 알림 서비스가 잠깐 꺼져 있어도 Kafka는 메시지를 보관함
   
2.2. kafka 만으로도 채팅 브로드캐스트가 가능할것같은데 Redis를 추가적으로 사용한이유
  - 실시간 브로드캐스트
    - Kafka는 지연이 있을 수 있지만 Redis는 휘발성이지만 빠름
    
  -  redis 는 유저별 채널이 생성되는 반면 kafka 는 topic에 모든알림이 들어오기때문에 성능저하 + 복잡성증가
  -  kafka 는 comsumer group 개념이 있어서 알림서비스가 scale out 됐을때 하나의 알림서비스로만 전달되므로, 일부유저는 알림을 못받음

2.3. mysql 대신 mongodb
   - 알림 데이터는 정형화 되지 않았고 단순 조회 & 저장이 핵심이기때문에 mongodb로 충분

추가적으로 고려하여 개발하면 좋은것
1. kafka 이벤트 중복 수신 방지
   - 이벤트에 고유한 id를 사용해서 중복체크
2. kafka 메시지 유실/지연에 대한 대비
- consumer 가 메시지를 읽은 후 처리 중 장애 발생 / consumer 가 처리 실패 / kafka 메시지 만료 등..
- Offset 수동 커밋 (Manual Commit) , Retry + Backoff 설정 , Dead Letter Topic (DLT) 구성
