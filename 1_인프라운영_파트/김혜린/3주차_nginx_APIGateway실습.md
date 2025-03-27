3주차 todolist
--- 

- [x]  nginx API Gateway 어플리케이션 개발 실습 및 테스트
    - GET/POST API 만들고, 종류에 따라 was1, was2 로 분기
    - 리버스 프록시 설정 (단기목표 1번)
- [x]  spring boot 로 API Gateway 어플리케이션 개발 실습 및 테스트
    - GET/POST API 만들고 호출해보기
- [ ]  JWT 인증 방법에 대한 이해
    - 인증된 사람만 API 를 사용할 수 있게 하려면?
- [x]  Spring gateway 라우팅 변경
----

## nginx API Gateway 어플리케이션 개발 실습 및 테스트

리버스 프록시 설정
- Client → nginx → 웹 서버로 접속할 수 있도록 내부 서버를 구축하는 것.
- nginX 가 사용자의 요청을 전달하기 때문에 직접 연결보다 부하를 줄여줄 수 있다.
- nginx 컨테이너에서 내부 서버 8081, 8082 로 client 요청을 전달하여, nginx 서버에서 리버스 프록시 역할을 할 수 있도록 한다.

```
구조

[클라이언트 요청] → [NGINX (8080)] → [Spring Boot (8081, 8082)]
```

nginx를 리버스 프록시로 사용한다면, 클라이언트는 포트 8080 번에 요청하고,
수신한 Nginx는 내부적으로 띄운 Spring boot application 8081, 8082번 포트로 전달하는 구조가 됨.

### nginx.conf
```
events { }

http {

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass "http://host.docker.internal:8081";  # 호스트의 8081번 포트로 전달. localhost로하면 안됨
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }

        location /user {
                    proxy_pass "http://host.docker.internal:8082/";  # 호스트의 8082번 포트로 전달. localhost로하면 안됨
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection 'upgrade';
                    proxy_set_header Host $host;
                    proxy_cache_bypass $http_upgrade;
                }
    }
}
```

8080번 포트를 사용하는 nginx 서버를 도커로 띄워서 스프링 부트로 생성한 8081,8082번 서버로 전달하는 것 확인 
스프링 어플리케이션에서는 System.setProperty("spring.config.name", "application-8082"); 로 프로퍼티 설정함

controller 소스 
- 메소드에 따른 동작 확인 
```java
package com.example.demo2.controllers;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/")
public class ByeController {
@GetMapping         // GET method
public String bye() {
return "Bye World!";
}

    @PostMapping()
    public String postData(@RequestBody String data) {
        return "Received 2: " + data;
    }
}
```


## spring cloud gateway 로 프록시 설정

---
### 구조 

── main/            
│   │   ├── java/com/example/           
│   │   │   ├── app8081/   # 8081번 포트에서 실행될 애플리케이션          
│   │   │   │   ├── App8081Application.java         
│   │   │   │   ├── controller/                 
│   │   │   │   │   ├── HelloController8081.java                
│   │   │   │   ├── service/                
│   │   │   │   ├── repository/             
│   │   │   ├── app8082/   # 8082번 포트에서 실행될 애플리케이션              
│   │   │   │   ├── App8082Application.java                 
│   │   │   │   ├── controller/             
│   │   │   │   │   ├── HelloController8082.java                
│   │   │   │   ├── service/                
│   │   │   │   ├── repository/             
│   │   │   ├── apigateway/   # API Gateway        
│   │   │   │   ├── ApiGatewayApplication.java   
│   ├── resources/    
│   │   ├── application.yml  # api gateway          
│   │   ├── application-8081.yml  # 8081번 애플리케이션 설정             
│   │   ├── application-8082.yml  # 8082번 애플리케이션 설정                 
│── pom.xml



### yaml 

---- 
api gateway yml 파일
```
# spring cloud gateway yml 파일
spring:
  main:
    web-application-type: reactive
  cloud:
    config:
      enabled: false
    gateway:
      routes:
        - id: service-1
          uri: http://localhost:8081
          predicates:
            - Path=/ # 기본 요청을 Spring App 1로 전달
        - id: service-2
          uri: http://localhost:8082
          predicates:
            - Path=/demo2 # `/post` 요청을 Spring App 2로 전달
          filters:
            - StripPrefix=1 # `/post`를 제거하고 내부 요청을 `/`로 보냄

server:
  port: 8080 # API Gateway가 실행될 포트

```

application yml 파일 

```
server:
  port: 8082

spring:
  main:
    web-application-type: reactive
  cloud:
    config:
      enabled: false
  profiles:
    active: default
```


