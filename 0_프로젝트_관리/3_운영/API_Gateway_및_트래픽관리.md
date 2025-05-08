## API Gateway 및 트래픽관리


API Gateway는 클라이언트가 각 마이크로서비스의 API를 직접 호출하지 않고, 중앙의 API Gateway를 통해 접근할 수 있도록 합니다. 
이를 통해 단일 진입점을 제공하며, 마이크로서비스 간의 요청 라우팅 및 트래픽 관리를 중앙에서 처리할 수 있습니다.

### 주요 기능
* 요청 라우팅: 클라이언트의 요청을 적절한 마이크로서비스로 전달.
* 로드 밸런싱: 여러 서비스 인스턴스에 대한 부하 분산.
* API 제한: 요청 수, 속도 제한 등 관리.
* 모니터링 및 로깅: API 호출에 대한 모니터링과 로깅을 통한 문제 진단.
<br>

### 사용 기술 스택
#### ✅ API Gateway (Spring Cloud Gateway)
Spring Cloud Gateway는 클라이언트 요청을 여러 마이크로서비스로 라우팅하는 API Gateway입니다.     
즉, 사용자가 한 곳(게이트웨이)으로 요청을 보내면 알아서 적절한 마이크로서비스로 연결해줍니다.    
마이크로서비스 구조를 감추고, 단일 진입점을 제공하여 모든 요청을 한 곳에서 관리할 수 있도록 합니다.     

#### ✅ JWT 토큰 전달 (Authentication Filter)
클라이언트의 요청 헤더에 포함된 JWT 토큰을 Auth 서비스로 전달하여 요청이 유효한 사용자로부터 온 것인지 인증합니다.    
토큰이 유효할 경우 인증 정보를 내부 서비스에 전달하며, 유효하지 않은 경우 요청을 차단합니다.   

#### ✅ Rate Limiting + Fallback (Redis Rate Limiter)
Redis 기반의 Token Bucket 알고리즘을 이용해 API 요청 수를 제한합니다.     
과도한 요청을 제어하여 시스템을 보호하고, Redis 장애 시에도 fallback 처리를 통해 서비스가 완전히 중단되지 않도록 합니다.     
사용자 ID 또는 IP 기준으로 요청을 식별하여 유연하게 제한 정책을 적용할 수 있습니다.   

<br>

### API Gateway 포트 
| 서비스 | path | 포트 (외부:내부) |
| --- | --- | --- |
| API GW | / | 8080:8080 |
| grafana | /grafana | 3000:3000 |
| 멤버 서비스  | /api/v1/members | 8082:8082 |
| oAuth 서비스  | /api/v1/auth | 8082:8082 |
| 질문 서비스  | /api/v1/questions |8081:8081 |

* 서비스 포트는 모든 도메인에 다 동일하게 적용
* API 별 path는 서비스 개발 팀에서 올려준 docs 참고함. 변경 시 이 문서도 업데이트 예정 
 
<br>

### v1.0  
<details>
 <summary>docker compose 배포를 위한 전달 사항</summary> 
### docker compose 요청 사항 
#### ✅ grafana 추가 사항 
```
environment:
  - GF_SERVER_ROOT_URL=https://dev.mumulbo.com/grafana/
  - GF_SERVER_SERVE_FROM_SUB_PATH=true 
```
grafana는 최상위 경로에서 login으로 redirect 하고 있기 때문에, mumulbo API GW 에서 /grafana로 진입 시 mmb의 login path 로 리다이렉트 됩니다.         
이를 방지하기 위해 grafana 내부에서 진행하도록 grafana 컨테이너 environment 에서 위 설정값을 세팅해줘야 합니다. 

#### ✅ container name 
api gateway 에 라우팅 규칙 정할 때 컨테이너 이름으로 라우팅되기 때문이 이 부분에 대한 땅땅땅이 필요함 

| 서비스 | container name | 
| --- | --- |
| API GW | api-gateway (이부분은 사실 상관없음요.) |
| 프론트엔드 | frontend-service |
| 멤버 서비스 | member-service |
| 질문 서비스 | question-service |
| 그라파나 | grafana (그라파나는 위 env 설정을 못해서 테스트를 못해봐서.. 코드에는 추가 안되어있음. 수정예정) |
| oAuth ?  | ..모름 |

#### ✅ .env 옵션    
API GW 에서는 local / dev / prod에 대한 구분만 필요합니다. 이부분은 서비스파트의 필요사항과 중복되는 것 같아서 이후에 옵션값으로 수정할 예정입니다.     
before 
```
server:
  port: 8443
```
after
```
server:
  port: ${APPLICATION_PORT}
```
</details>

### v1.1  
<details>
 <summary> 모니터링 서비스 지원 및 Rate limiting </summary> 

## 1. grafana 라우팅 및 모니터링 API 지원
### 개요
mmb v1.0 에서는 api gateway가 grafana의 컨테이너로 라우팅이 되지 않는 문제가 있어서, 현재는 api 로 제공되지 않고 서비스 외부 포트로 클라이언트가 접근하고 있음. 
grafana의 리다이렉션 처리에 대한 현상으로 추정되며, v1.1 기능에 모니터링 서비스에 대한 API 라우팅을 지원한다. 

#### 현상 설명 
```
# grafana 컨테이너 docker compose 
environment:
  - GF_SERVER_ROOT_URL=https://dev.mumulbo.com/grafana/
  - GF_SERVER_SERVE_FROM_SUB_PATH=true 
```

위 옵션을 주고 /grafana 로 접근 시 [https://dev.mumulbo.com/login?redirectTo=%2Fgrafana](https://dev.mumulbo.com/login?redirectTo=%2Fgrafana) 형태로 리디렉션됨. 이는 로그인 과정이 그라파나 로그인이 아닌 무물보 로그인으로 실행되는 것임.  
Gateway의 " RewritePath=/grafana(?/?.*), /${segment} " 도 선언해주었으나 지속적으로 발생하여, 우선은 SERVER_ROOT_URL 을 localhost:3000 으로 선언해서 그라파나 컨테이너의 단독 실행에는 문제가 발생하지 않도록 함 ...  
  
또한 /grafana/login 으로 접근 시 아래 그라파나 프록시 설정에러 페이지가 나옴  
[![intro](https://github.com/A-OverFlow/mmb-docs/raw/main/9_images/hlkim_5_3.png)](https://github.com/A-OverFlow/mmb-docs/blob/main/9_images/hlkim_5_3.png)

### 계획 
- 그라파나의 접근은 grafana.mumulbo.com 으로 진행한다. 
- 모니터링 어플리케이션 repo pull 받아서 로컬 환경에서 api gw -> grafana 컨테이너로 라우팅 확인할 예정 
- 라우팅 정상 동작 확인 후, 리다이렉션 문제 확인. (프론트엔드/모니터랑 담당과 협업 예정)

### 기대 목표 
- 정상으로 라우팅되어 mumulbo 도메인에서 grafana 서비스를 조회할 수 있도록 한다. 


## 2. Rate Limiting 기능 추가 
API Gateway(Spring Cloud Gateway)에서 일정 시간 내 과도한 요청을 차단하여 **시스템 안정성 및 보안성 향상**을 목표로 한다. 
    
### RedisRateLimiter 
요청 제한 기능은 Redis 기반의 Token Bucket 알고리즘을 활용하여 기능을 개발한다.      
Redis는 중앙 캐시로서 작동하므로, 여러 대의 Gateway 인스턴스가 동일한 요청 제한 기준을 공유할 수 있어 **스케일 아웃 환경에서도 일관된 제어가 가능**하다.    
Token Bucket 방식은 요청을 일정 속도로 처리하면서도, 순간적인 트래픽 증가(버스트)에 유연하게 대응할 수 있는 구조로, **서비스 안정성과 사용자 경험을 동시에 고려한 제한 방식**이다.

#### 제한 정책 
- 제한 기준 (셋 중에 하나로 진행 예정)
	- IP 기준 (유력): IP KeyResolver 로 IP 기준으로 트래픽 제한 예정 
	- User ID 기준: 사용자 단위로 도배 제한 
	- IP + User ID 기준 (Custom KeyResolver) : 사용자명 없으면 IP로 fallback. 로그인 유저는 ID 기준으로,, 비로그인 유저는 IP 기준으로 
- Rate Limit Token
	- replenishRate: 10  (1초에 10개 허용) 
	- burstCapacity: 20 (몰릴 경우 20개까지 허용가능)
- 차단 시 응답 
	- HTTP 429 Too Many Requests

### 계획
- redis 연결을 위해 redis 의존성 컨테이너 추가 (기존 auth가 사용하고 있는 redis 와 별개)
- RequestRateLimiter 필터 추가



### 적용 대상 
- Gateway를 통해 접근하는 모든 API 엔드포인트 (/api/**)

```
# application.yml 예시 

filters:
  - name: RequestRateLimiter
    args:
      redis-rate-limiter.replenishRate: 10
      redis-rate-limiter.burstCapacity: 20
      key-resolver: "#{@hybridKeyResolver}"

```

## 3. Fallback 클래스 추가 
Redis를 이용한 Rate limiting 기능은 모든 요청의 처리 여부를 redis에 의존하는 형태임.      
즉, Redis의 장애나 미 연결시 요청을 받아도 제한 판단에 대한 오판이 이루어질 수 있음. 

이를 방지하기 위해 적절한 에러 메시지를 추가하거나 Fallback 응답을 통해 예외 전파를 하고, 모니터링서비스와 연계하여 상황을 전달해야함.   

### 계획 
- redis 연결 실패 시 IP 기준으로 간단하게 막기 
- 로그로 에러 전파 (onErrorResume . .)
- API 경로 별 fallback 허용/차단 판별 (redis가 없어도 fallback으로 허용할 수 있는 서비스는? )

```
return redisRateLimiter.isAllowed(id, routeId).onErrorResume(e -> {
    log.error("Redis 연결 실패로 RateLimit 판단 불가: {}", e.getMessage());
    return Mono.just(new Response(false, -1));
});

```


### 필터 우선 순위 

| 필터명 | 우선순위 | 비고 |
| --- | --- | --- |
| LoggingFilter | -1 | - |
| RateLimiter | - | Route Filter 라 우선순위 설정 X. (커스텀 필터로 사용할 경우 GatewayFilterFactory 필터). Route Filter 보다 우선적으로 실행될 필터는 음수로 우선순위를 둠. |
| JwtAuthenticationFilter | 10 | - |
| PostRoutingLoggingFilter | 10100 | - |

</details>

 
