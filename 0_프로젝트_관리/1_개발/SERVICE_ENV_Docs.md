# 1. 서비스 별 사용 환경 변수

## 1.1. 공통 환경 변수

| 환경변수            | 설명           | 비고 |
|-----------------|--------------|----|
| EUREKA_HOST     | 서비스 디스커버리 주소 |    |
| EUREKA_PORT     | 서비스 디스커버리 포트 |    |
| PROMETHEUS_HOST | 서비스 디스커버리 주소 |    |
| PROMETHEUS_PORT | 서비스 디스커버리 포트 |    |

## 1.2. 회원 서비스

| 환경변수                     | 설명                 | 비고               |
|--------------------------|--------------------|------------------|
| APPLICATION_NAME         | 서비스 이름             |                  |
| APPLICATION_PORT         | 서비스 포트             |                  |
| DATASOURCE_URL           | 데이터베이스 URL         |                  |
| DATASOURCE_USERNAME      | 데이터베이스 사용자         |                  |
| DATASOURCE_PASSWORD      | 데이터베이스 패스워드        |                  |
| REDIS_URL                | REDIS URL          |                  |
| REDIS_PORT               | REDIS 포트           |                  |
| SECRET_KEY               | JWT 서명용 KEY        | 최소 32글자 이상       |
| REFRESH_TOKEN_EXPIRATION | refresh token 만료일자 | MILLI_SECONDS 단위 |
| ACCESS_TOKEN_EXPIRATION  | access token 만료일자  | MILLI_SECONDS 단위 |

## 1.3. 질문/답변 서비스

| 환경변수                | 설명           | 비고 |
|---------------------|--------------|----|
| APPLICATION_PORT    | 서비스 포트       |    |
| DATASOURCE_URL      | 데이터 베이스 URL  |    |
| DATASOURCE_USERNAME | 데이터 베이스 사용자  |    |
| DATASOURCE_PASSWORD | 데이터 베이스 패스워드 |    |
