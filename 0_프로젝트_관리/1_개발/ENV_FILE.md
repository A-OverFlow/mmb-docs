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
    # 방법 1. env파일을 사용
    env_file:
      - .env

    # 방법 2. 개별 맵핑, 실행시 --env-file 옵션 사용 필요
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

도커 컴포즈 내에서 `env_file` 같이 파일 경롤 설정하는 방법도 지원  
다수의 .env파일 사용시 같은 값이면 마지막 파일에 값으로 overwrite됨  
환경 변수가 많다면 파일 env파일 설정 하는 방법 사용  

환경 변수가 많지 않다면 개별 맵핑을 하여 사용하는 방법 사용  
서비스 별 편리한 방법을 통해 구동하도록 한다  

실행 명령어
```bash
docker-compose --env-file .env
```
실행시에 .env파일 경로 설정을 통해 구동 가능

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
