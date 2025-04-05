# 무물보 환경 변수 가이드 문서

## 1. 개발 가이드

application.yml 파일

```yaml
server:
  port: {MEMBER_SERVICE_PORT}

...

  application:
    name: {MEMBER_SERVICE_NAME}

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: {MYSQL_URL}
    username: {MYSQL_USERNAME}
    password: {MYSQL_PASSWORD}

...

jwt:
  secret : {SECRET}
  access-token-expire-time : {ACCESS_TOKEN_EXPIRE_TIME}
  refresh-token-expire-time : {REFRESH_TOKEN_EXPIRE_TIME}
```

.env
```bash
MEMBER_SERVICE_PORT=8110
MEMBER_SERVICE_NAME=member-service
MYSQL_URL=jdbc:mysql://localhost:3306/aof?serverTimezone=Asia/Seoul
MYSQL_USERNAME=root
MYSQL_PASSWORD=root1234
SECRET=9cd1de305ba49cd1c5bd1e3d626ddd0eadabcc4316948e808782025b65cc36ee
ACCESS_TOKEN_EXPIRE_TIME=300
REFRESH_TOKEN_EXPIRE_TIME=7200
```

docker-compose.yml
```
    env_file:
      - .env
    environment:
      - MEMBER_SERVICE_NAME=${MEMBER_SERVICE_NAME}
      - MEMBER_SERVICE_PORT=${MEMBER_SERVICE_PORT}
      - MYSQL_URL=${MYSQL_URL}
      - MYSQL_USERNAME=${MYSQL_USERNAME}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - SECRET=${SECRET}
      - ACCESS_TOKEN_EXPIRE_TIME=${ACCESS_TOKEN_EXPIRE_TIME}
      - REFRESH_TOKEN_EXPIRE_TIME=${REFRESH_TOKEN_EXPIRE_TIME}
```

실행 명령어
```bash
docker-compose --env-file .env
```

설정 확인
```bash
docker-compose config
```

## 0. 공통 환경 변수

|환경 변수 이름|값|설명|
|------|------|------|
|EUREKA_SERVER_URL|1.1.1.1|유레카 서버 URL|
|EUREKA_SERVER_PORT|8888|유레카 서버 PORT|
|MEMBER_SERVICE_NAME|member-service|회원 서비스 이름|
|MEMBER_SERVICE_PORT|8080|회원 서비스 포트|
|QUESTION_SERVICE_NAME|question-service|회원 서비스 이름|
|QUESTION_SERVICE_PORT|8181|질문/답변 서비스 포트|

### 2. 서비스 별 환경 변수

#### 2.1. MEMBER-SERVICE

|환경 변수 이름|값|설명|
|------|------|------|
|MYSQL_URL|1.1.1.1|회원 서비스 Mysql URL|
|MYSQL_USERNAME|root|회원 서비스 Mysql 계정|
|MYSQL_PASSWORD|root1234|회원 서비스 Mysql 패스워드|
|REDIS_HOST|1.1.1.2|Redis 서버 주소|
|REDIS_PORT|6739|Redis 서버 포트|

#### 2.2. QUESTION-SERVICE

|환경 변수 이름|값|설명|
|------|------|------|
|MYSQL_URL|2.2.2.2|질문/답변 서비스 Mysql URL|
|MYSQL_USERNAME|root|질문/답변 서비스 Mysql 계정|
|MYSQL_PASSWORD|root1234|질문/답변 서비스 Mysql 패스워드|
|KAFKA_URL|3.3.3.3|Kafka의 주소|
