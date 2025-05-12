> 무물보짱의 Docker Compose 에 관한 내용과 공유 사항을 기록합니도

## docker-compose.yml

### 설명

* 컨테이너 서비스 별로 배포에 필요한 옵션을 docker-compose로 옮겼습니다.
* V2부터는 version 명시가 필요없대서(채찍기니가 그럼) 안썼습니다.
* **networks**: 각 서비스별로 필요한 망에만 연결했습니다. db는 외부 (프론트엔드)에서 직접 접근을 못하게 분리했습니다.

  * fe-net: gw, fe
    * 외부 유저 요청을 받음
  * be-net: service
    * 백엔드 서비스 간 내부 통신
  * db-net: redis, mysql
    * db와 캐시 서버에 대한 내부 통신 전용 네트워트
  * mgt-net: 모니터링
    * 모니터링 전용 관리 망
* **depends_on** :

  * mysql, redis → member, question service → gw → fe
  * node, process exporter → prometheus → grafana
  * ✅원래는 먼저 실행되어야 할 서비스의 헬스체크 후 실행하려 했으나, service쪽 actuator dependency 추가 작업이 필요해 일단은 헬스체크 없는 버전
    * **추후 헬스체크 추가 예정**
  
### .env
.env.dev, .env.prod 파일을 이용해 Docker compose 실행 시 환경 별 변수를 분리해 관리합니다.

프로젝트 루트 경로에 docker compose, .env.{환경} 파일이 위치
#### .env.dev
```
MYSQL_URL=jdbc:mysql://localhost:3306/aof?serverTimezone=Asia/Seoul
MYSQL_USERNAME=root
MYSQL_PASSWORD=root1234

# 포트 설정 - 내부는 고정(8080), 외부는 개발용 포트
MEMBER_SERVICE_PORT=9082
QUESTION_SERVICE_PORT=9081
```

#### .env.prod
```
MYSQL_URL=jdbc:mysql://localhost:3306/aof?serverTimezone=Asia/Seoul
MYSQL_USERNAME=root
MYSQL_PASSWORD=root1234

# 포트 설정 - 내부는 고정(8080), 외부는 개발용 포트
MEMBER_SERVICE_PORT=8082
QUESTION_SERVICE_PORT=8081
```


### 환경 별 docker compose 실행 명령
```
# 개발 환경
docker compose --env-file .env.dev up -d

# 운영 환경
docker compose --env-file .env.prod up -d
```
* **env** :

    * .env.dev, .env.prod 로 분기
    * ✅미니멀 배포 시점엔 mysql 접속 정보만 넣기로 협의.
        * 비즈니스파트 측에서 서비스에 말아서 배포되는 application.yml에 나머지 환경변수들 하드코딩 하기로함
        * .env.{환경}, application.yml(단일) 사용

<details> <summary><strong>✅ docker-compose.yml</strong></summary>

```yml
services :
  mysql:
    image: mysql:latest
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root1234
      TZ: Asia/Seoul
    ports:
      - 3306:3306
    volumes:
      - ./db/mysql/data:/var/lib/mysql
    platform: linux/x86_64
    networks:
      - db-net

  redis:
    image: redis:latest
    container_name: redis
    restart: always
    ports:
      - 6379:6379
    volumes:
      - ./redis/data:/data
      - ./redis/conf/redis.conf:/usr/local/conf/redis.conf
    labels:
      - "name=redis"
      - "mode=standalone"
    command: redis-server /usr/local/conf/redis.conf
    networks:
      - db-net

  member-service:
    image: member-service
    container_name: member-service
    restart: unless-stopped
    depends_on:
      - mysql
      - redis
    env_file:
      - .env
    environment:
      - MYSQL_URL=${MYSQL_URL}
      - MYSQL_USERNAME=${MYSQL_USERNAME}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    ports:
      - "${MEMBER_PORT}:8080"
    command:
      - # ㅁㄹ
    networks:
      - fe-net
      - be-net
      - db-net

  question-service:
    image: question-service
    container_name: question-service
    restart: unless-stopped
    depends_on:
      - mysql
      - redis
      - member-service
    env_file:
      - .env
    environment:
      - MYSQL_URL=${MYSQL_URL}
      - MYSQL_USERNAME=${MYSQL_USERNAME}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    ports:
      - "${QUESTION_PORT}:8080"
    command:
      - # ㅁㄹ
    networks:
      - fe-net
      - be-net
      - db-net

  gateway:
    image: api-gateway
    container_name: api-gateway
    restart: unless-stopped
    depends_on:
      - member-service
      - question-service
    environment:
      - APPLICATION_PORT=${APPLICATION_PORT}
    networks:
      - fe-net

  mmb-frontend:
    image: myeongseob91/mmb-frontend:latest
    container_name: mmb-frontend
    restart: unless-stopped
    depends_on:
      - gateway
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /home/ec2-user/infra/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt:ro
    command: ["nginx", "-g", "daemon off;"]
    networks:
      - fe-net

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    networks:
      - mgt-net
      - be-net

  process-exporter:
    image: ncabatoff/process-exporter:latest
    container_name: process-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /etc:/host/etc:ro
    command: [ "—procfs=/host/proc" ]
    ports:
      - "9256:9256"
    networks:
      - mgt-net
      - be-net

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: unless-stopped
    depends_on:
      - node-exporter
      - process-exporter
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - mgt-net

  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: unless-stopped
    depends_on:
      - prometheus
    environment:
      - GF_SERVER_ROOT_URL=https://dev.mumulbo.com/grafana/
      - GF_SERVER_SERVE_FROM_SUB_PATH=true
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - "3000:3000"
    networks:
      - mgt-net

networks :
  fe-net:
    driver: bridge
  be-net:
    driver: bridge
  db-net:
    driver: bridge
  mgt-net:
    driver: bridge
```
</details>