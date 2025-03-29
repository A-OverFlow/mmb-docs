## API Gateway 및 트래픽관리

----
API Gateway는 클라이언트가 각 마이크로서비스의 API를 직접 호출하지 않고, 중앙의 API Gateway를 통해 접근할 수 있도록 합니다. 
이를 통해 단일 진입점을 제공하며, 마이크로서비스 간의 요청 라우팅 및 트래픽 관리를 중앙에서 처리할 수 있습니다.

### 주요 기능
* 요청 라우팅: 클라이언트의 요청을 적절한 마이크로서비스로 전달.
* 로드 밸런싱: 여러 서비스 인스턴스에 대한 부하 분산.
* API 제한: 요청 수, 속도 제한 등 관리.
* 모니터링 및 로깅: API 호출에 대한 모니터링과 로깅을 통한 문제 진단.

### 사용 기술 스택

#### ✅ API Gateway (Spring Cloud Gateway)
Spring Cloud Gateway는 클라이언트 요청을 여러 마이크로서비스로 라우팅하는 API Gateway입니다.
즉, 사용자가 한 곳(게이트웨이)으로 요청을 보내면 알아서 적절한 마이크로서비스로 연결해줍니다.
마이크로서비스 구조를 감추고, 단일 진입점을 제공하여 모든 요청을 한 곳에서 관리할 수 있도록 합니다. 


#### ✅ Eureka (Spring Cloud Eureka) : 서비스 디스커버리
- Eureka Server: 모든 마이크로서비스를 등록하고 관리하는 서버
- Eureka Client: Eureka Server에 자신을 등록하고, 다른 서비스 정보를 가져가는 클라이언트

- 디스커버리 : Spring Cloud Eureka <br>

#### ✅ Config server (Spring Cloud Config ) : 설정 관리
Spring Cloud Config는 모든 마이크로서비스의 설정을 한 곳에서 관리할 수 있도록 도와주는 서비스입니다.
환경별(dev, prod) 설정을 따로 관리 가능하고, 설정 변경 시 애플리케이션을 재시작하지 않고 반영 가능합니다. 
무물보에서는 git이나 DB에서 관리 예정 


### App 별 포트 (todo)
| Service             | 포트번호    |
|---------------------|---------|
| API GW              | 8000    |
| eureka              | 8761    |
| eureka client       | 8070 .. |
| cloud config server | 8888    |
| member service      | 8010    |
| question service    | 8100    |
| ...                 | ....    |


### 개발 계획
1. Gateway Application 프로젝트 생성
2. eureka 추가 (server, client) 
2. cloud config client (새로운 repo 생성)
3. cloud config server 
5. Spring cloud bus 

### 배포 및 운영
- 배포 계획: Spring Cloud Gateway는 Kubernetes, Docker, 또는 클라우드 환경에 배포할 수 있습니다.
- 운영 계획: 모니터링 및 로깅 도구를 사용하여 API 호출의 성능을 추적하고, 장애를 관리합니다.




 