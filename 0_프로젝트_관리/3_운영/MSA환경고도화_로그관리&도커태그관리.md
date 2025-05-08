## MSA 환경 고도화 작업 문서

### 1. 개선 필요성

* **로그 수집의 한계점**

    * 기존에는 `docker logs` 명령어를 통해 로그를 확인했으나:

        * 컨테이너 재시작 시 로그 유실
        * 로그 검색 및 장기 보존이 어려움
        * 외부 로그 수집 도구와의 연동이 어려움

* **Docker 이미지 태그 충돌 문제**

    * `dev` 고정 태그를 사용해 이미지 덮어씌움 발생
    * 변경 추적이 어려워 디버깅과 롤백이 어려움

---

### 2. 로그 수집 구조 개선

* **Logback 파일 기반 로깅 적용**

    * `logback-spring.xml`을 활용해 파일 로그 저장 방식으로 전환
    * Spring Profile은 사용하지 않고 `LOG_PATH` 환경 변수 존재 여부로 경로 판단

* **Docker Volume 마운트 구조**

    * docker-compose.yml 설정 예시:

      ```yaml
      volumes:
        - /home/ec2-user/deploy/mmb-ms-msa-playground/logs:/app/logs
      ```
    * 해당 경로로 로그가 EC2 로컬 파일 시스템에 저장됨
    *

* **기대 효과**

    * 로그 파일을 안정적으로 보존 가능
    * Loki / Promtail 등 로그 수집기 연동 기반 마련
    * 운영자와 개발자가 동일한 위치에서 로그 확인 가능

---

### 3. Docker 이미지 태그 전략 개선

* **기존 문제점**

    * `dev` 고정 태그로 인해 이미지가 매번 덮어씌워짐
    * 이전 버전으로의 롤백 불가
    * 어떤 커밋이 배포되었는지 추적 어려움

* **새로운 태그 포맷 도입**

    * 형식: `dev-<날짜>-<SHA7>` 예: `dev-20240507-abc1234`
    * 장점:

        * 배포일자 + Git 커밋 기반으로 추적 용이
        * 이미지 중복 방지 및 안정적인 롤백 가능

* **docker-compose.generated.yml이란?**

    * CI/CD 파이프라인에서는 매 배포 시점마다 고유한 이미지 태그가 사용되므로,
      기존의 `docker-compose.yml`을 그대로 사용할 수 없음
    * 따라서 원본 compose 파일에는 `IMAGE_TAG_PLACEHOLDER`라는 임시 문자열을 넣어두고,
      실제 배포 시점에 CI에서 `sed` 명령어로 이를 실제 태그(ex. dev-20240507-abc1234)로 치환함
    * 치환 결과물로 생성되는 파일이 `docker-compose.generated.yml`이며, 해당 파일이 EC2 서버로 전달되어 실행됨
    * 이렇게 하면 매 배포마다 별도의 태그가 적용된 compose 파일이 생성되어 안정성과 추적성을 확보할 수 있음

* **배포 순서도**

  ```
  1. GitHub Actions 시작
      ↓
  2. docker-compose.yml에는 아직 실제 이미지 태그가 없음
     - 예: IMAGE_TAG_PLACEHOLDER 라는 자리 표시자 존재
      ↓
  3. IMAGE_TAG 생성
     - 예: dev-20240507-abc1234
      ↓
  4. sed 명령어 실행
     - docker-compose.yml에서 "IMAGE_TAG_PLACEHOLDER" 를 실제 태그로 치환
     - 결과: docker-compose.generated.yml 생성됨
      ↓
  5. EC2 서버로 복사
     - docker-compose.generated.yml → EC2:/배포경로/docker-compose.yml
      ↓
  6. EC2에서 docker compose up -d 실행
     - docker-compose.generated.yml의 내용으로 컨테이너 실행
  ```

* **GitHub Actions 설정 요약**

    * 주요 단계:

        1. `TAG="dev-$(date +%Y%m%d)-${GITHUB_SHA::7}"`로 고유 태그 생성
        2. `sed` 명령어로 원본 compose 파일을 복사 및 치환 → `docker-compose.generated.yml` 생성
        3. `.env` 및 생성된 compose 파일을 EC2에 복사
        4. EC2 내에서 `docker compose up -d` 명령어 실행
        5. 불필요한 이미지가 쌓이지 않도록 최근 10개를 제외한 이전 이미지 정리

---

### 4. 소스 코드

(생략: logback-spring.xml, docker-compose.yml, cd-dev.yml)

```yml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true">

<!-- LOG_PATH가 지정되지 않으면 기본값으로 ./logs 사용 -->
<property name="LOG_PATH" value="${LOG_PATH:-./logs}" />
<property name="LOG_FILE_NAME" value="app.log" />

<!-- 콘솔 로그 -->
<appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
<encoder>
<pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
</appender>

<!-- 파일 로그 -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<file>${LOG_PATH}/${LOG_FILE_NAME}</file>
<encoder>
<pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
</encoder>
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
<fileNamePattern>${LOG_PATH}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}.gz</fileNamePattern>
<maxHistory>30</maxHistory>
</rollingPolicy>
</appender>

<root level="INFO">
<appender-ref ref="CONSOLE" />
<appender-ref ref="FILE" />
</root>

</configuration>

```

```yml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: mumulbo/mmb-ms-msa-playground:IMAGE_TAG_PLACEHOLDER
    env_file:
      - .env
    ports:
      - "${SERVICE_PORT}:8084"
    volumes:
      - /home/ec2-user/deploy/mmb-ms-msa-playground/logs:/app/logs
    networks:
      - external-net
    depends_on:
      - db
```

```yml
name: DEV CD - Deploy to EC2

# 워크플로우 트리거 조건 설정
on:
  # dev 브랜치에 Push 발생 시 실행
  push:
    branches: [ develop ]

jobs:
  deploy-dev:
    # 사용할 머신 환경 - 최신 Ubuntu LTS
    runs-on: ubuntu-latest
    # 사용할 환경(dev)의 Secrets와 Variables 를 활성화
    environment: dev

    steps:
      - name: 소스 코드 체크아웃
        uses: actions/checkout@v4

      - name: Docker Buildx 설정 (멀티 플랫폼 빌드를 위해 사용)
        uses: docker/setup-buildx-action@v3

      - name: EC2 접속을 위한 SSH 키 설정
        run: |
          mkdir -p ~/.ssh                    # .ssh 디렉터리 생성
          echo "${{ secrets.DEV_EC2_KEY }}" > ~/.ssh/ec2.pem  # 비밀키 파일 생성
          chmod 600 ~/.ssh/ec2.pem            # 비밀키 파일 권한 설정

      - name: 저장소 이름 및 배포 디렉토리 변수 설정
        id: vars
        run: |
          echo "REPO_NAME=${GITHUB_REPOSITORY##*/}" >> $GITHUB_OUTPUT  # 저장소 이름 추출
          echo "DEPLOY_DIR=${{ secrets.DEV_DEPLOY_PATH }}/${GITHUB_REPOSITORY##*/}" >> $GITHUB_OUTPUT  # 배포 경로 설정

      - name: 이미지 태그 생성 (ex:dev-20240505-abc1234)
        id: tagger
        run: |
          TAG="dev-$(date +%Y%m%d)-${GITHUB_SHA::7}"
          echo "IMAGE_TAG=$TAG" >> $GITHUB_OUTPUT

      - name: Docker Hub 로그인
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Docker 이미지 빌드 후 Docker Hub에 푸시
        uses: docker/build-push-action@v5
        with:
          context: .               # 빌드 컨텍스트 (현재 디렉토리)
          file: Dockerfile          # 사용할 Dockerfile
          push: true                # 빌드 후 이미지 푸시
          tags: mumulbo/mmb-ms-msa-playground:${{ steps.tagger.outputs.IMAGE_TAG }}  # 이미지 태그 설정

      - name: docker-compose.yml에 이미지 태그 반영
        run: |
          sed "s|IMAGE_TAG_PLACEHOLDER|${{ steps.tagger.outputs.IMAGE_TAG }}|g" docker-compose.yml > docker-compose.generated.yml

      - name: EC2 서버에 배포 디렉토리 생성
        run: |
          ssh -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no ${{ secrets.DEV_EC2_HOST }} \
          "mkdir -p ${{ steps.vars.outputs.DEPLOY_DIR }}"

      - name: docker-compose.yml 파일 EC2 서버로 복사
        run: |
          scp -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no docker-compose.generated.yml \
          ${{ secrets.DEV_EC2_HOST }}:${{ steps.vars.outputs.DEPLOY_DIR }}/docker-compose.yml

      - name: 배포에 필요한 .env 파일 생성
        run: |
          echo "SERVICE_PORT=${{ vars.SERVICE_PORT }}" > .env
          echo "DB_NAME=${{ vars.DB_NAME }}" >> .env
          echo "DB_HOST=${{ vars.DB_HOST }}" >> .env
          echo "HOST_DB_PORT=${{ vars.HOST_DB_PORT }}" >> .env
          echo "CONTAINER_DB_PORT=${{ vars.CONTAINER_DB_PORT }}" >> .env
          echo "DB_USERNAME=${{ secrets.DB_USERNAME }}" >> .env
          echo "DB_PASSWORD=${{ secrets.DB_PASSWORD }}" >> .env
          echo "DB_ROOT_PASSWORD=${{ secrets.DB_ROOT_PASSWORD }}" >> .env
          echo "LOG_PATH=${{ vars.LOG_PATH }}" >> .env

      - name: 생성한 .env 파일 EC2 서버로 복사
        run: |
          scp -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no .env \
          ${{ secrets.DEV_EC2_HOST }}:${{ steps.vars.outputs.DEPLOY_DIR }}/.env

      - name: EC2 서버에서 docker-compose를 사용해 배포
        run: |
          ssh -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no ${{ secrets.DEV_EC2_HOST }} << EOF
            cd ${{ steps.vars.outputs.DEPLOY_DIR }}
            docker compose up -d   # 컨테이너를 백그라운드로 실행
          EOF

      - name: EC2 서버에서 오래된 Docker 이미지 정리 (최신 10개만 보존)
        run: |
          ssh -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no ${{ secrets.DEV_EC2_HOST }} << 'EOF'
            docker images --format '{{.Repository}}:{{.Tag}} {{.CreatedAt}}' \
              | grep 'mumulbo/mmb-ms-msa-playground:dev-' \
              | sort -r \
              | awk 'NR > 10 { print $1 }' \
              | xargs -r docker rmi
          EOF
```

---

### 5. 작업 스크린샷

![w9_ms_msaplus_locallog.png](..%2F..%2F9_images%2Fw9_ms_msaplus_locallog.png)
![w9_ms_msaplus_locallogview.png](..%2F..%2F9_images%2Fw9_ms_msaplus_locallogview.png)
![w9_ms_msaplus_action_variable.png](..%2F..%2F9_images%2Fw9_ms_msaplus_action_variable.png)
![w9_ms_msaplus_dockerhub.png](..%2F..%2F9_images%2Fw9_ms_msaplus_dockerhub.png)
![w9_ms_msaplus_ec2dockercomposeview.png](..%2F..%2F9_images%2Fw9_ms_msaplus_ec2dockercomposeview.png)
![w9_ms_msaplus_ec2logview.png](..%2F..%2F9_images%2Fw9_ms_msaplus_ec2logview.png)
