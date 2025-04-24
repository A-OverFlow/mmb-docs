작성기한: 2025-04-24

# 7주차 - Spring Boot 설정파일(`.yml`) 관리하기


*목차*
* `1) application.yml 작성법`
* `2) application.yml 동작 원리`
* `3) application.yml 관리법`
  * `1. 환경별 설정 분리 (application-{profile}.yml)`
  * `2. 환경변수 사용하기`
    * `(1) .env 파일을 개발자가 직접 작성해서 사용`
    * `(2) IntelliJ의 환경 변수 메뉴를 활용`
* `4) 그렇다면 무물보 프로젝트에서는?`

---

Spring Boot로 애플리케이션의 설정파일을 작성하고 관리하는 방법을 알아봅시다.

우리는 보통 Spring Boot로 애플리케이션을 만들 때 주요 설정값들을 관리하기 위해<br/>
`application.properties` 또는 `application.yml`와 같은 설정파일을 작성합니다.<br/>
(두 형식 간 차이가 궁금하다면 [이 글](https://www.baeldung.com/spring-boot-yaml-vs-properties) 참고)

이런 설정 파일을 어떻게 작성하는지 살펴 봅시다.<br/>
본 문서에선 `application.yml` 파일 중심으로 서술합니다.

<br/><br/>

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

<br/><br/>

## 2) `application.yml` 동작 원리
이게 어케 가능한 걸까요? Spring Boot의 설정파일 동작 원리를 살펴 봅시다.

1. Spring Boot가 시작되면<br/>
→ application.yml 파일을 자동으로 로드함.

2. YAML 파일의 계층 구조를 분석해서<br/>
→ 각각의 값을 Environment와 @Value, @ConfigurationProperties 등에 매핑함.

3. 필요한 설정 값들을 기반으로<br/>
→ 내장 톰캣, DB 연결, JPA, 로깅 등 다양한 Spring Bean들을 자동 구성함.


그러니까 위에 적은 예제로 치면 아래와 같이 동작하게 되는 것이지요
* `server.port`: 내장 톰캣 서버가 **8080** 포트에서 실행됨.
* `spring.datasource.*`: Spring Boot가 **DataSource 객체를 구성**할 때 사용됨.
  * 이 설정을 기반으로 JDBC 연결이 이루어지고, 필요시 JPA/Hibernate도 설정함.

<br/><br/>

## 3) `application.yml` 관리법
작성법과 동작원리는 알겠습니다. 그렇다면 생산적인 개발 환경을 위해<br/>
`application.yml` 같은 설정파일을 어떻게 관리해야 할까요?

<br/>

### 1. 환경별 설정 분리 (`application-{profile}.yml`)
로컬, 개발, 운영 등 환경을 나눠서 설정하고 싶다면<br/>
다음과 같이 설정파일의 프로파일(profile)을 구분해 관리할 수 있습니다.

`application.yml`
```yml
spring:
  profiles:
    active: local  # 현재 활성화할 profile 지정
```

`application-local.yml` (h2 사용 예시)
```yml
server:
  port: 8080

spring:
   datasource:
     url: jdbc:h2:tcp://localhost/~/myapp
     username: sa
     password:
     driver-class-name: org.h2.Driver
 
   jpa:
     hibernate:
       ddl-auto: create
     properties:
       hibernate:
         show_sql: true
         format_sql: true
 
   logging:
     level:
       org.hibernate.SQL: debug
       org.hibernate.orm.jdbc.bind: trace
```

`application-dev.yml` (MySQL 사용 예시)
```yml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/devdb
    username: devuser
    password: devpassword
    # ...
```

`application-prod.yml` (MySQL 사용 예시)
```yml
server:
  port: 80

spring:
  datasource:
    url: jdbc:mysql://prod-server:3306/proddb  # 그러나 prod의 민감정보는 이 예시처럼 로컬에서 작성해서는 안 되고 그 대신 GitHub Secrets 등 플랫폼을 활용함.
    username: produser
    password: prodpassword
    # ...
```
이렇게 프로파일을 나눠서 작성하면 `spring.profiles.active` 값에 따라<br/>
특정 프로파일이 알아서 애플리케이션에 적용됩니다.

그러니까 `spring.profiles.active` 값이 `local`이면 `application-local.yml`에서 설정한 값이,<br/>
`dev`이면 `application-dev.yml`에서 설정한 값이 적용된다는 겁니다.

<br/>

### 2. 환경변수 사용하기
사실 `datasource`의 `url`, `username`, `password`처럼 민감한 정보들은<br/>
문자열 그대로 넣으면 안 되고, 그 대신 환경변수를 활용해야 합니다.

예를 들어 `application-dev.yml` 파일에는 각 값에 문자열 대신 다음과 같이 환경변수를 적용해야 합니다.

`application-dev.yml`
```yml
server:
  port: 8080

spring:
  datasource:
    url: ${DEV_DATASOURCE_URL}
    username: ${DEV_DATASOURCE_USERNAME}
    password: ${DEV_DATASOURCE_PASSWORD}
    # ...
```

`*환경변수를 설정*`
```
DEV_DATASOURCE_URL=jdbc:mysql://localhost:3306/devdb
DEV_DATASOURCE_USERNAME=devuser
DEV_DATASOURCE_PASSWORD=devpassword
```

위에 `*환경변수를 설정*`이라고 애매하게 작성한 이유는 몰까요.<br/>
환경변수를 설정하는 방법에는 여러가지가 있기 때문입니다.
1. `.env` 파일을 개발자가 직접 작성해서 사용
2. IntelliJ의 환경 변수 메뉴를 활용
3. Docker로 확장해서 구성해 사용... 등

이중에서 1번과 2번 방법을 알아 봅시다. 3번은 Docker 토픽 문서에서 참조하시죠

#### **(1) `.env` 파일을 개발자가 직접 작성해서 사용**
로컬에서 `.env` 파일을 직접 작성하는 예제를 살펴 봅시다.<br/>
`local`과 `dev` 프로파일별로 `.env` 파일을 분리하고 싶다고 가정합니다.

일단 프로젝트 루트 디렉토리에 `.env.local` 파일, `.env.dev`파일을 각각 생성하고<br/>
아래처럼 작성합니다. (그리고 `.gitignore`에 추가하는 것을 잊지 않습니다)

`.env.local`
```dotenv
LOCAL_DATASOURCE_URL=*로컬 DB URL 정보*
LOCAL_DATASOURCE_USERNAME=*로컬 DB 유저네임*
LOCAL_DATASOURCE_PASSWORD=*로컬 DB 비밀번호*
```
`.env.dev`
```dotenv
DEV_DATASOURCE_URL=*개발 DB URL 정보*
DEV_DATASOURCE_USERNAME=*개발 DB 유저네임*
DEV_DATASOURCE_PASSWORD=*개발 DB 비밀번호*
```


그리고 각각의 `.yml` 파일에 환경변수를 적용합니다.

`src/main/resources/application-local.yml`
```yml
spring:
  datasource:
    url: ${LOCAL_DATASOURCE_URL}
    username: ${LOCAL_DATASOURCE_USERNAME}
    password: ${LOCAL_DATASOURCE_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

`src/main/resources/application-dev.yml`
```yml
spring:
  datasource:
    url: ${DEV_DATASOURCE_URL}
    username: ${DEV_DATASOURCE_USERNAME}
    password: ${DEV_DATASOURCE_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

그 다음, IntelliJ에서 `.env` 파일 주입을 설정합니다. 이때 플러그인이 필요함니다.

🔹 EnvFile 플러그인 설치
1. IntelliJ → File > Settings > Plugins
2. EnvFile 검색 → 설치 → 재시작

🔹 실행 설정에 `.env.local`, `.env.dev` 연결
1. IntelliJ 메뉴: Run > Edit Configurations...
2. 실행 대상 (예: DemoApplication) 선택
3. 아래에 EnvFile 체크박스 활성화
5. `+` 클릭 → `.env.local`, `.env.dev` 파일 선택

![image](https://github.com/user-attachments/assets/2414cfde-5ef7-4893-b42c-fdc5160e0df0)

이러면 끝.


#### **(2) IntelliJ의 환경 변수 메뉴를 활용**
아니면 걍 IntelliJ에서 GUI로 제공하는 환경 변수 메뉴를 활용할 수도 있습니다.
1. IntelliJ 메뉴: Run > Edit Configurations...
2. Environmental variables의 달러 기호 버튼 클릭
3. 필요한 환경변수들 복붙 > OK

![image](https://github.com/user-attachments/assets/6d576d94-70d1-4bf7-a49b-1db706eef58c)

이렇게도 가능. 끝.

<br/><br/>

## 4) 그렇다면 무물보 프로젝트에서는?
그렇다면 비즈니스 파트 서비스에서는 어케 관리할까요<br/>
현재로선 서비스 내에선 환경 분리 없이 하나의 `application.yml` 파일을 아래 예시와 같이 설정했으며<br/>
변수화해 관리하는 정보들은 다음과 같습니다. 이제 환경 분리가 필요합니다~
* `server.port`
* `spring.application.name`
* `spring.datasource.url`
* `spring.datasource.username`
* `spring.datasource.password`

<br/>

Question Service `application.yml` 예시:
```yml
server:
  port: ${QUESTION_SERVICE_PORT}
  servlet:
    encoding:
      charset: UTF-8
      enabled: true
      force: true

spring:
  application:
    name: ${QUESTION_SERVICE_NAME}

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: ${DATASOURCE_URL}
    username: ${DATASOURCE_USERNAME}
    password: ${DATASOURCE_PASSWORD}

  jpa:
    hibernate:
      ddl-auto: none
    properties:
      hibernate:
        format_sql: false
    open-in-view: false
    show-sql: false

logging:
  level:
    org.hibernate.SQL: debug
    org.hibernate.orm.jdbc.bind: off
```


끝.
