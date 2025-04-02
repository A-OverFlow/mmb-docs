## 1. ec2 폴더 구성

```
/home/ec2-user/infra/
├── docker-compose.yml
├── mysql/
├──── docker-compose.yml
├──── db
├── redis/
├──── docker-compose.yml
├──── redis
└── nginx

```

## 2. github action 설정

> 자동화된 배포로 독립적인 서비스 관리가 가능하게 합니다.
>
> github action을 통해 cicd 파이프라인 구성을 하기 위해 필요한 파일 :
> 1. Dockerfile
> 2. deploy.yml
>
> Docker Image를 Docker Hub에 저장하고, 서버에서 pull하기 때문에 Docker Hub 계정이 필요합니다.
> 계정의 경우, 공용으로 관리하는 것이 좋습니다.

### 1. Dockerfile

CI/CD 파이프라인에서 애플리케이션을 컨테이너화 하는 역할을 합니다.

예시 :

```
# OpenJDK 21 기반 이미지 사용 (JDK 21 맞춰서 변경)
FROM openjdk:21-jdk

# 컨테이너 내에서의 작업 디렉토리 설정
WORKDIR /app

# JAR 파일 복사 (Gradle 빌드 경로에 맞게 설정)
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar

# 컨테이너에서 실행할 명령어
CMD ["java", "-jar", "app.jar"]
```

### 2. deploy.yml

deploy.yml은 GitHub Actions에서 배포(Deploy)를 자동화하는 Workflow 파일입니다.
deploy.yml .github/workflows/ 경로에 위치하며, CI/CD에서 배포 단계를 정의하는 역할을 수행합니다.

위에서 언급한 Dockerfile로 컨테이너화 한 이미지를 실행합니다.

예시 :

```
name: Deploy Config Server to EC2  # GitHub Actions 워크플로우 이름

on:
  push:
    branches:
      - main  # main 브랜치에 push가 발생하면 실행

jobs:
  deploy:
    runs-on: ubuntu-latest  # 배포 작업을 수행할 환경 (Ubuntu 최신 버전 사용)

    steps:
      - name: Checkout source code
        uses: actions/checkout@v3  # 현재 레포지토리의 코드를 체크아웃 (다운로드)

      - name: Set up JDK 21
        uses: actions/setup-java@v3  # Java 21 설치
        with:
          java-version: '21'
          distribution: 'temurin'  # Temurin JDK 사용

      - name: Grant execute permission to Gradle
        run: chmod +x ./gradlew  # Gradle 실행 권한 부여

      - name: Build with Gradle
        run: ./gradlew clean build  # Gradle을 사용하여 프로젝트 빌드

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
        # Docker Hub 로그인 (비밀번호는 GitHub Secrets에서 가져옴)

      - name: Build Docker image
        run: docker build -t myeongseob91/mmb-config-server .  # Docker 이미지 빌드

      - name: Push Docker image
        run: docker push myeongseob91/mmb-config-server  # Docker Hub에 빌드된 이미지 업로드

      - name: Deploy on EC2
        uses: appleboy/ssh-action@v0.1.6  # SSH를 사용하여 원격 서버에 배포
        with:
          host: ${{ secrets.EC2_HOST }}  # EC2 서버의 IP 주소
          username: ${{ secrets.EC2_USER }}  # EC2 로그인 사용자 이름
          key: ${{ secrets.EC2_KEY }}  # SSH 키 (GitHub Secrets에서 가져옴)
          script: |
            sudo docker pull myeongseob91/mmb-config-server  # 최신 Docker 이미지 가져오기
            sudo docker stop app || true  # 기존 컨테이너 중지 (없어도 오류 무시)
            sudo docker rm app || true  # 기존 컨테이너 삭제 (없어도 오류 무시)
            sudo docker run -d --name config-server -p 8888:8888 myeongseob91/mmb-config-server  # 새 컨테이너 실행
```

---

## 3. frontend 배포

### **1.1. frontend 소스코드 추가**

(1) deploy.yml

```yaml
name: Deploy Frontend via Docker

on:
  push:
    branches:
      - develop

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. 소스코드 체크아웃
      - name: Checkout code
        uses: actions/checkout@v3

      # 2. Node.js 설치
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      # 3. 의존성 설치
      - name: Install dependencies
        run: npm install

      # 4. 프로젝트 빌드
      - name: Build project
        run: npm run build

      # 5. 빌드 결과 확인
      - name: Check build output
        run: ls -l ./dist

      # 6. Docker 이미지 빌드
      - name: Build Docker image
        run: |
          sudo docker build -t ${{ secrets.DOCKER_USERNAME }}/mmb-frontend:latest .

      # 7. Docker Hub 로그인
      - name: Log in to Docker Hub
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | sudo docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      # 8. Docker 이미지 푸시
      - name: Push Docker image
        run: |
          sudo docker push ${{ secrets.DOCKER_USERNAME }}/mmb-frontend:latest

      # 9. EC2에 배포
      - name: Deploy on EC2
        env:
          EC2_HOST: ${{ secrets.EC2_HOST }}
          EC2_KEY: ${{ secrets.EC2_KEY }}
          EC2_USER: ${{ secrets.EC2_USER }}
          DOCKER_IMAGE: ${{ secrets.DOCKER_USERNAME }}/mmb-frontend:latest
        run: |
          echo "${{ secrets.EC2_KEY }}" > /tmp/key.pem
          chmod 400 /tmp/key.pem
          ssh -o StrictHostKeyChecking=no -i /tmp/key.pem $EC2_USER@$EC2_HOST "
            sudo docker pull $DOCKER_IMAGE && \
            sudo docker stop mmb-frontend || true && \
            sudo docker rm mmb-frontend || true && \
            sudo docker run -d -p 80:80 -p 443:443 --name mmb-frontend $DOCKER_IMAGE
          "

      # 10. 이메일 알림 (성공 여부에 따라)
      - name: Send Email Notification
        if: always()
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: ${{ secrets.SMTP_HOST }}
          server_port: ${{ secrets.SMTP_PORT }}
          username: ${{ secrets.SMTP_USER }}
          password: ${{ secrets.SMTP_PASSWORD }}
          subject: "MMB Frontend Deployment - ${{ job.status }}"
          body: |
            The MMB frontend deployment via GitHub Actions has completed.
            Status: ${{ job.status }}
            Branch: ${{ github.ref }}
            Commit: ${{ github.sha }}
            Run ID: ${{ github.run_id }}
          to: ${{ secrets.RECIPIENT }}
          from: ${{ secrets.SMTP_USER }}
          secure: false
          ignore_cert: false

```

(2) nginx.conf

```yaml
worker_processes 1;

  events {
  worker_connections 1024;
}

  http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  sendfile on;
  keepalive_timeout 65;

  server {
  listen 80;
  server_name mumulbo.com www.mumulbo.com;
  return 301 https://$host$request_uri;
  }

  server {
  listen 443 ssl;
  server_name mumulbo.com www.mumulbo.com;

  ssl_certificate /etc/letsencrypt/live/mumulbo.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/mumulbo.com/privkey.pem;

  location / {
  root /usr/share/nginx/html;
  index index.html;
  try_files $uri $uri/ /index.html;
  autoindex on;  # 디렉터리 인덱스를 허용
  }

  # 추가적인 리디렉션 및 접근 제어
  location ~ /\.env {
  deny all;
  }
  }
}
```

(3) DockerFile

```dockerfile
# 1단계: 빌드
FROM node:18-alpine AS build
WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# 2단계: nginx로 배포
FROM nginx:alpine

# Nginx 기본 경로 생성
RUN mkdir -p /usr/share/nginx/html

# 빌드 결과 복사
COPY --from=build /app/dist /usr/share/nginx/html

# Nginx 설정 복사
COPY ./nginx/nginx.conf /etc/nginx/nginx.conf

CMD ["nginx", "-g", "daemon off;"]
```

## 4. nginx 설정 및 https 설정

(1) SSL 인증서 발급

- dnf install -y certbot
- certbot --version
- certbot certonly --standalone -d mumulbo.com
- certbot certificates

etc/letsencrypt/ 경로 하위에 인증서 생김 (갱신주기가 짧음 90일)

(2) docker-compose.yml

- docker-compose.yml 로 시작할때 ec2 에 있는 nginx 설정을 컨테이너 안으로 마운트시킴
- docker-compose.yml 로 시작할때 ec2 에 있는 ssl인증서를 컨테이너 안으로 마운트시킴

```yaml
services:
  mmb-frontend:
    image: myeongseob91/mmb-frontend:latest
    container_name: mmb-frontend
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /home/ec2-user/infra/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt:/etc/letsencrypt:ro
    command: [ "nginx", "-g", "daemon off;" ]
```