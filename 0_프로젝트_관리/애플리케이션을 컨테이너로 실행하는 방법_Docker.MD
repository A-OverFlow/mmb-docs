# 1. 각 언어(프레임워크)로 만들어진 애플리케이션을 컨테이너로 배포하는 방법 (Docker 기준)

![Container Deployment](./images/container_deployment.png)

## 1-1. Dockerfile 작성

`Dockerfile`이란 도커에서 이미지를 생성하기 위한 용도로 작성하는 설정 파일입니다.  
프로젝트 루트 디렉토리에 `Dockerfile` 파일을 생성하고 이미지를 생성하는데 필요한 명령어를 작성합니다.

1. 베이스 이미지 설정
2. 컨테이너 내 작업 디렉토리 설정
3. 실행 파일을 컨테이너 내부로 복사
4. 애플리케이션 실행 명령어 설정
5. 애플리케이션 포트 설정

### 예시) Spring Boot 애플리케이션에 대한 Dockerfile

``` dockerfile
# 1. 베이스 이미지를 'JDK 21' 기본 이미지로 사용
FROM openjdk:21-jdk

# 2. 컨테이너에서 작업 디렉토리를 '/A-OverFlow'로 지정
WORKDIR /A-OverFlow

# 3. target 폴더에 존재하는 'application.jar' 파일을 컨테이너 작업 디렉토리에 복사
COPY target/application.jar application.jar

# 4. 'application.jar' 파일 실행 명령어 설정
CMD ["java", "-jar", "application.jar"]

# 5. 애플리케이션을 실행할 포트를 8080 포트로 지정
EXPOSE 8080
```

Reference: https://docs.docker.com/reference/dockerfile/

## 1-2. Docker 이미지 빌드

현재 디렉토리에 있는 `Dockerfile` 파일을 이용하여 도커 이미지를 생성합니다.

```
docker build -t {image_name} .
```

References: https://docs.docker.com/build/concepts/dockerfile/#building

## 1-3. 컨테이너 실행

생성한 도커 이미지를 기반으로 컨테이너를 생성합니다.  
`--name` 옵션을 통해 컨테이너에 이름을 붙일 수도 있습니다.

```
docker run -d --name {container_name} {image_name}
```

References: https://docs.docker.com/engine/containers/run/

## 1-4. 실행 중인 컨테이너 확인

아래의 명령어를 통해 도커 컨테이너 목록과 상태를 확인합니다.

```
docker ps
```

![docker ps](./images/docker_ps.png)

References: https://docs.docker.com/reference/cli/docker/container/ls/

# 2. 애플리케이션을 컨테이너로 만드는 방법

# 3. 컨테이너 이미지를 관리하는 방법

# 4. 컨테이너를 배포하기 위해 설정해야 하는 서버 작업들

# 5. 애플리케이션을 컨테이너로 배포하면 좋은점

# 6. 컨테이너 안에서 생성되는 애플리케이션 로그를 확인하는 방법

## 6-1. 도커 기본 로그 확인

`docker logs` 명령어를 통해 도커 컨테이너의 표준 출력(stdout)과 표준 에러(stderr) 로그를 확인할 수 있습니다.  
`-f` 옵션을 통해 실시간으로 로그를 모니터링할 수 있습니다.

```
docker logs -f {container_name or container_id}
```

References: https://docs.docker.com/reference/cli/docker/container/logs/

## 6-2. 컨테이너 내부로 직접 접속하여 로그 확인

일부 애플리케이션은 컨테이너 내부의 특정 디렉토리에 로그 파일을 저장하기 때문에  
컨테이너 내부에서 해당 디렉토리에 접속하여 로그 파일을 확인할 수 있습니다.

```
docker exec -it {container_name or container_id} /bin/bash
cd /var/log
cat application.log
```

## 기타 방법으로 로그 확인

컨테이너 로그를 파일로 저장하거나 도커 컨테이너를 특정 폴더와 마운트시켜 로그 파일을 직접 확인할 수 있습니다.  
또한 컨테이너의 로그를 ELK, Fluentd, Loki와 같은 로그 수집 시스템과 연동하여 로그를 확인할 수 있습니다.
