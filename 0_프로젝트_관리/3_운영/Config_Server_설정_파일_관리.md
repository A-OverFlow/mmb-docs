# Config Server 란?

> **Config Server**는 마이크로서비스 아키텍처(MSA)에서 **각 서비스의 설정(Configuration)을 중앙에서 관리**하는 역할을 하는 서버

Spring Cloud Config Server 를 많이 사용

MSA에서는 각 서비스가 독립적으로 배포되고 실행되므로 **환경 설정 파일(application.properties, application.yml 등)을 한 곳에서 관리하는 게 중요**

## 왜 MSA에서는 Config Server 를 사용하는가?

#### 1. **설정의 중앙 관리**

* 기존 모놀리식(Monolithic)에서는 하나의 `application.yml` 파일을 관리하면 됨.
* 하지만 MSA에서는 **각 서비스마다 개별 설정이 필요**함.
* **Config Server를 사용하면 모든 마이크로서비스의 설정을 한 곳에서 관리 가능**.

#### 2. **환경별 설정 관리 (Development, Staging, Production)**

* `application-dev.yml`, `application-prod.yml` 같은 환경별 설정을 **Config Server에 저장**하고,
  서비스가 실행될 때 자동으로 적절한 설정을 가져올 수 있음.

#### 3. **서비스 배포 없이 설정 변경 가능**

* 일반적으로 설정을 변경하려면 서비스 재배포가 필요하지만,
  **Config Server를 사용하면 재배포 없이 설정 변경이 가능**함.
* 예를 들어, `config-repo`에서 설정 파일을 변경하면 각 서비스가 자동으로 최신 설정을 반영함.

#### 4. **보안 (민감한 정보 보호)**

* DB 비밀번호, API 키, 인증 정보 등 **민감한 설정 값**을 코드에 하드코딩하면 보안상 위험.
* **Config Server에 암호화된 상태로 저장하고, 필요할 때만 접근하도록 설정** 가능.

#### 5. **서비스 확장성**

* 여러 개의 서비스가 동일한 설정 값을 공유해야 할 때,
  Config Server를 통해 **일괄적으로 설정을 적용**할 수 있음.
* 새로운 서비스가 추가되어도 **중앙에서 설정을 관리하므로 쉽게 확장 가능**.

---

## Config Server에서 설정이 갱신되는 과정
### 1. Config Server에서 설정 제공
1. Config Server는 Git, 파일 시스템, DB 등에 저장된 설정을 관리함.
2. 서비스가 실행될 때 Config Server로부터 설정을 가져옴.
3. 서비스가 실행된 후에도 설정이 변경되면 최신 값을 반영할 수 있음.

### 2. 서비스가 설정을 불러오는 과정
#### Client Server
원래는 bootstrap.yml에 config server 정보를 저장했지만,

deprecated된 관계로 appliaction.yml에 config server 정보를 적어넣는다.
```
spring:
    config:
        import: 
            optional:
                configserver: http://localhost:8888
```

### 3. 설정 변경 시 갱신되는 과정
1. 설정 파일을 관리하는 깃에 수정된 설정 파일을 푸시한다.
    2. 저장소에서 설정이 업데이트되면 Config Server는 새로운 설정을 제공할 준비를 함.
    3. 하지만 변경된 설정이 즉시 반영되지는 않음! (서비스는 자동으로 인식하지 않음)

2. 반영하는 방법에는 3가지가 있는데,
    3. 애플리케이션 재시작 (서비스 중단이 발생할 수 있으니 운영 환경에서는 비추)
    4. /actuator/refresh 엔드포인트 호출 ( hot reload, 무중단 적용)
        5. application.yml에 actuator를 활성화해야함
        6. 서비스 설정 최신 반영을 위해 /actuator/refresh로 post 요청을 보냄
    7. **Spring Cloud Bus** 👍
        8. kafka, rabbitMQ를 통해 모든 서비스에 자동 전파 가능
        9. 특정 서비스에서 /actuator/bus-refresh/ 를 한번만 호출하면, 모든 인스턴스에 변경 사항이 자동으로 전파됨

—
## 무물보 Client Server가.. 할 일…
1. application.yml에 cloud bus 사용 정보 입력
2. 설정 파일은 설정 파일 레포에 푸시
3. 특정 인스턴스에서 bus-refresh 요청