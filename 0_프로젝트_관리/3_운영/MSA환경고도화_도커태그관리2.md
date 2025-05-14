1. main branch 에 태그 생성
   ![w10_ms_msaplus_tagcreate.png](..%2F..%2F9_images%2Fw10_ms_msaplus_tagcreate.png)
2. develop 브랜치에 태그 merge 및 push

3. develop 브랜치 에서 git describe 명령어 정상 수행되는것 확인
   ![w10_ms_msaplus_gitdiscribe.png](..%2F..%2F9_images%2Fw10_ms_msaplus_gitdiscribe.png)
4. cd-dev.yml 수정

```yml
name: DEV CD - Deploy to EC2

on:
  push:
    branches: [ develop ]

jobs:
  deploy-dev:
    runs-on: ubuntu-latest
    environment: dev

    steps:
      - name: 소스 코드 체크아웃
        uses: actions/checkout@v4
        with:
          # 아래 옵션을줘야 tag 까지 checkout
          fetch-depth: 0
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

      - name: Docker 이미지 빌드 및 푸시 (정확한 태그 + dev-latest)
        uses: docker/build-push-action@v5
        with:
          context: .
          file: Dockerfile
          push: true
          # IMAGE_TAG 붙여서 push
          # dev-latest push 
          # 2가지 이미지를 푸쉬해서 dev-latest 태그륿 붙이면 항상 최신것이 다운되도록함
          tags: |
            mumulbo/mmb-ms-msa-playground:${{ steps.tagger.outputs.IMAGE_TAG }}
            mumulbo/mmb-ms-msa-playground:dev-latest

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
              | grep 'mumulbo/mmb-ms-msa-playground:v' \
              | sort -r \
              | awk 'NR > 10 { print $1 }' \
              | xargs -r docker rmi
          EOF

```

![w10_ms_msaplus_tag.png](..%2F..%2F9_images%2Fw10_ms_msaplus_tag.png)
![w10_ms_msaplus+dockerhub.png](..%2F..%2F9_images%2Fw10_ms_msaplus%2Bdockerhub.png)
![w10_ms_msaplus_latest.png](..%2F..%2F9_images%2Fw10_ms_msaplus_latest.png)

필요한 명령어들

태그 확인하기
git tag

원격 태그 가져오기
git fetch --tags

태그가 가리키는 커밋과 커밋 메시지, 파일 변경 내역까지 모두 보여줌
git show v1.0.0