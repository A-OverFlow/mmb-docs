작성기한: 2025-04-24

# 7주차 - Spring Boot 설정파일(`.yml`) 관리하기

Spring Boot로 애플리케이션의 설정파일을 작성하고 관리하는 방법을 알아봅시다.

우리는 보통 Spring Boot로 애플리케이션을 만들 때<br/>
주요 설정값들을 관리하기 위해 `application.properties`<br/>
또는 `application.yml`와 같은 설정파일을 작성합니다.<br/>
(두 형식 간 차이가 궁금하다면 [이 글](https://www.baeldung.com/spring-boot-yaml-vs-properties) 참고)

이런 설정 파일을 어떻게 작성하는지 살펴 봅시다.<br/>
본 문서에선 `application.yml` 파일 중심으로 서술합니다.


## 1) `application.yml` 작성법
### 1. 기본 구조(예시)
Spring Boot는 다음과 같이 작성한 `application.yml`를 읽어와<br/>
해당 설정값들을 애플리케이션 실행 시 알아서 적용합니다.
```yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: myuser
    password: mypassword
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  ...
```


## 2) `application.yml` 동작 원리
이게 어케 가능한 걸까요? Spring Boot의 설정파일 동작 원리를 살펴 봅시다.

1. Spring Boot가 시작되면
→ application.yml 파일을 자동으로 로드함.

2. YAML 파일의 계층 구조를 분석해서
→ 각각의 값을 Environment와 @Value, @ConfigurationProperties 등에 매핑함.

3. 필요한 설정 값들을 기반으로
→ 내장 톰캣, DB 연결, JPA, 로깅 등 다양한 Spring Bean들을 자동 구성함.


그러니까 위에 적은 예제로 치면 아래와 같이 동작하게 되는 것
* `server.port`: 내장 톰캣 서버가 **8081** 포트에서 실행됨.
* `spring.datasource.*`: Spring Boot가 **DataSource 객체를 구성**할 때 사용됨.
  * 이 설정을 기반으로 JDBC 연결이 이루어지고, 필요시 JPA/Hibernate도 설정함.


## 3) `application.yml` 관리법
동작 원리도 알겠고 작성법도 알겠습니다. 그렇다면 생산적인 개발 환경을 위해 우리는 `application.yml` 같은 설정파일을 어떻게 관리해야 할까요?

### 1. 환경별 설정 분리 (`application-{profile}.yml`)
개발, 운영 환경을 나눠 설정하고 싶다면 다음과 같이 설정파일의 프로파일을 구분해 관리할 수 있습니다.

`application.yml`
```yml
spring:
  profiles:
    active: dev  # 현재 활성화할 profile 지정
```

`application-dev.yml`
```yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/devdb
    username: devuser
    password: devpassword
```

`application-prod.yml`
```yml
server:
  port: 80

spring:
  datasource:
    url: jdbc:mysql://prod-server:3306/proddb
    username: produser
    password: prodpassword
```
이렇게 작성하면 `spring.profiles.active` 값에 따라,<br/>
`application-dev.yml` 또는 `application-prod.yml` 설정이<br/>
알아서 애플리케이션에 적용됩니다.

### 2. 환경변수 사용하기
사실 `url`, `username`, `password`처럼 민감한 정보들은 문자열 그대로 넣으면 안 되고, 그 대신 환경변수를 활용해야 합니다.
```
SPRING_DATASOURCE_URL=jdbc:mysql://prod-server:3306/proddb
SPRING_DATASOURCE_USERNAME=produser
SPRING_DATASOURCE_PASSWORD=securepassword
```

.env 파일 사용 (Docker, 일부 로컬 툴)하거나, IntelliJ 환경 변수 설정 방법이 있음

## 4) 그렇다면 무물보 프로젝트에서는?
무물보에서는 각 서비스마다 포트번호도 DB도 다르니 다음과 같이 환경변수들도 달리해야겠지요,,,
* 회원 서비스(Member Service)
  * 개발/운영 `spring.datasource.*`, `server.port`, `spring.application.name`...
* Auth 서비스(Auth Service)
  * 개발/운영 `spring.datasource.*`, `server.port`, `spring.application.name`...
* 질문 서비스(Question Service)
  * 개발/운영 `spring.datasource.*`, `server.port`, `spring.application.name`...

끝.
