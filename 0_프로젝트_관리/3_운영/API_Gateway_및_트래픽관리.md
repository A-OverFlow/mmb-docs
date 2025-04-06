## API Gateway 및 트래픽관리


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


#### ✅ Config server (Spring Cloud Config ) : 설정 관리
Spring Cloud Config는 모든 마이크로서비스의 설정을 한 곳에서 관리할 수 있도록 도와주는 서비스입니다.
환경별(dev, prod) 설정을 따로 관리 가능하고, 설정 변경 시 애플리케이션을 재시작하지 않고 반영 가능합니다. 
무물보에서는 git이나 DB에서 관리 예정 


### API Gateway 포트 
#### 목적 별 포트 
| 목적 | 설명 | 파일 | 포트 |
| --- | --- | --- | --- |
| 로컬 개발 | all 도메인 + http | application-local.yml |  8080 |
| 개발 서버  | 서브 도메인 + https | application-dev.yml | 8443 |
| 운영 서버 | 메인 도메인 + https | application-prod.yml | 443 |
* 위에서 설정해주는 포트는 각 yml 파일들에서 오버라이드되기 때문에 docker compose의 port 에서는 운영용 포트만 선언해주시면 됩니다. 


#### 서비스별 포트 
* 서비스 포트는 서브도메인/메인도메인 동일하게 사용함

| 서비스 | path | 포트 (외부:내부) |
| --- | --- | --- |
| grafana | /grafana | 3000:3000 |
| 멤버 서비스  | /api/v1/member | 8082:8082 |
| 질문 서비스  | /api/v1/questions |8081:8082 |
| 프론트 엔드  | / | 81:80 |
| Eureka  | 운영용 | 8761 (todo) |
| config server  | 운영용 | todo |

* API 별 path는 서비스 개발 팀에서 올려준 docs 참고함. 변경 시 이 문서도 업데이트 예정 
 

### docker compose 요청 사항 

#### grafana 추가 사항 
```
environment:
  - GF_SERVER_ROOT_URL=https://dev.mumulbo.com/grafana/
  - GF_SERVER_SERVE_FROM_SUB_PATH=true 
```
grafana는 최상위 경로에서 login으로 redirect 하고 있기 때문에, mumulbo API GW 에서 /grafana로 진입 시 mmb의 login path 로 리다이렉트 됩니다.         
이를 방지하기 위해 grafana 내부에서 진행하도록 grafana 컨테이너 environment 에서 위 설정값을 세팅해줘야 합니다. 


### 배포 및 운영
- docker compose 환경에서 
- 배포 계획: Spring Cloud Gateway는 Kubernetes, Docker, 또는 클라우드 환경에 배포할 수 있습니다.
- 운영 계획: 모니터링 및 로깅 도구를 사용하여 API 호출의 성능을 추적하고, 장애를 관리합니다.




 
