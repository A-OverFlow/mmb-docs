### 도커 태그 설정

1. main 브랜치에서 git tag -a v0.0.1 -m "Initial tag"
2. develop 브랜치 git merge main main을 develop에 머지(충돌있을경우 해결해야함)
3. develop 브랜치 git describe 나오는것 확 (ex : v0.0.1-36-gaea434f)
4. cd-dev.yml 수정 ("여기" 로 검색해보세요. 수정해야하는부분 주석달아놨습니다.)

```yml
name: Dev CD

on:
  pull_request:
    types: [ closed ]
    branches:
      - develop
  push:
    branches: [ develop ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: dev

    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        #여기
        with:
          fetch-depth: 0

      - name: Docker Buildx 설정 (멀티 플랫폼 빌드를 위해 사용)
        uses: docker/setup-buildx-action@v3

      - name: EC2 접속을 위한 SSH 키 설정
        run: |
          mkdir -p ~/.ssh                                         # .ssh 디렉터리 생성
          echo "${{ secrets.DEV_EC2_KEY }}" > ~/.ssh/ec2.pem      # 비밀키 파일 생성
          chmod 600 ~/.ssh/ec2.pem                                # 비밀키 파일 권한 설정

      - name: 저장소 이름 및 배포 디렉토리 변수 설정
        id: vars
        run: |
          echo "REPO_NAME=${GITHUB_REPOSITORY##*/}" >> $GITHUB_OUTPUT 
          echo "DEPLOY_DIR=${{ secrets.DEV_DEPLOY_PATH }}/${GITHUB_REPOSITORY##*/}" >> $GITHUB_OUTPUT
      #여기
      - name: Git 태그 기반 이미지 태그 생성
        id: tagger
        # IMAGE_TAG 값을 git describe 를 활용해서 만듬
        run: |
          git fetch --tags
          TAG=$(git describe --tags --match "v[0-9]*" --abbrev=7 2>/dev/null || echo "dev-unknown-${GITHUB_SHA::7}")
          echo "IMAGE_TAG=$TAG" >> $GITHUB_OUTPUT

      - name: Docker Hub 로그인
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      #여기
      - name: Docker 이미지 빌드 후 Docker Hub에 푸시
        uses: docker/build-push-action@v5
        with:
          context: .
          file: Dockerfile
          push: true
          # IMAGE_TAG 붙여서 push
          # dev-latest push
          # 2가지 이미지를 푸쉬해서 dev-latest 태그륿 붙이면 항상 최신것이 다운되도록함
          tags: |
            mumulbo/mmb-apigateway:${{ steps.tagger.outputs.IMAGE_TAG }}
            mumulbo/mmb-apigateway:dev-latest
      #여기
      - name: docker-compose.yml에 이미지 태그 반영
        run: |
          sed "s|IMAGE_TAG_PLACEHOLDER|${{ steps.tagger.outputs.IMAGE_TAG }}|g" docker-compose.yml > docker-compose.generated.yml

      - name: EC2 서버에 배포 디렉토리 생성
        run: |
          ssh -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no ${{ secrets.DEV_EC2_HOST }} \
          "mkdir -p ${{ steps.vars.outputs.DEPLOY_DIR }}"
      #여기
      - name: docker-compose.yml 파일 EC2 서버로 복사
        run: |
          scp -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no docker-compose.generated.yml \
          ${{ secrets.DEV_EC2_HOST }}:${{ steps.vars.outputs.DEPLOY_DIR }}/docker-compose.yml

      - name: 배포에 필요한 .env 파일 생성
        run: |
          > .env
          echo "APIGATEWAY_PORT=${{ vars.APIGATEWAY_PORT }}" >> .env
          echo "APIGATEWAY_NAME=${{ vars.APIGATEWAY_NAME }}" >> .env
          echo "DOCKER_IMAGE=${{ vars.DOCKER_IMAGE }}" >> .env
          echo "MMB_DOCKER_NETWORK=${{ vars.MMB_DOCKER_NETWORK }}" >> .env
          echo "JWT_SECRET_KEY=${{ secrets.JWT_SECRET_KEY }}" >> .env
          echo "HOST_DOMAIN=${{ vars.HOST_DOMAIN }}" >> .env
          echo "QUESTION_SERVICE_PORT=${{ vars.QUESTION_SERVICE_PORT }}" >> .env
          echo "MEMBER_SERVICE_PORT=${{ vars.MEMBER_SERVICE_PORT }}" >> .env
          echo "AUTH_SERVICE_PORT=${{ vars.AUTH_SERVICE_PORT }}" >> .env
          echo "FRONTEND_PORT=${{ vars.FRONTEND_PORT }}" >> .env
          echo "GRAFANA_PORT=${{ vars.GRAFANA_PORT }}" >> .env
          echo "CHAT_SERVICE_PORT=${{ vars.CHAT_SERVICE_PORT }}" >> .env

      - name: 생성한 .env 파일 EC2 서버로 복사
        run: |
          scp -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no .env \
          ${{ secrets.DEV_EC2_HOST }}:${{ steps.vars.outputs.DEPLOY_DIR }}/.env

      - name: EC2 서버에서 docker-compose를 사용해 배포
        run: |
          ssh -i ~/.ssh/ec2.pem -o StrictHostKeyChecking=no ${{ secrets.DEV_EC2_HOST }} << EOF
            cd ${{ steps.vars.outputs.DEPLOY_DIR }}
            #여기
            docker compose down
            docker compose up -d   # 컨테이너를 백그라운드로 실행
          EOF

```

5. docker-compose.yml 수정

```yml
services:
  api-gateway:
    #여기
    image: ${DOCKER_IMAGE}:IMAGE_TAG_PLACEHOLDER
    container_name: mmb-apigateway
    ports:
      - "${APIGATEWAY_PORT}:${APIGATEWAY_PORT}"   # 외부 8080 -> 내부 8080
    env_file:
      - .env
    networks:
      - mumulbo-network
    restart: no

networks:
  mumulbo-network:
    external: true
    driver: bridge
    name: ${MMB_DOCKER_NETWORK}

```

### 로그 설정

1.docker-compose.yml 수정

```yml
services:
  api-gateway:
    image: ${DOCKER_IMAGE}:IMAGE_TAG_PLACEHOLDER
    container_name: mmb-apigateway
    ports:
      - "${APIGATEWAY_PORT}:${APIGATEWAY_PORT}"   # 외부 8080 -> 내부 8080
    env_file:
      - .env
    networks:
      - mumulbo-network
    restart: no
    #여기
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"

networks:
  mumulbo-network:
    external: true
    driver: bridge
    name: ${MMB_DOCKER_NETWORK}

```