> 서비스(도커 이미지, 컨테이너)를 사용하는 방법에 대해서 기록합니다.

> 아래 목차는 예시이며 작성자 임의로 목록을 추가하거나 수정해서 사용할 수 있습니다.

# Sample Service

- 서비스 설명

## Docker 이미지 정보

- 이미지: `ghcr.io/your-org/sample-service`
- 태그: `latest`, `v1.0.0`

## 실행 방법

```bash
docker run -d \
  --name user-service \
  -e SPRING_PROFILES_ACTIVE=prod \
  -p 8080:8080 \
  ghcr.io/your-org/sample-service:latest
````

## 환경변수

* `SPRING_PROFILES_ACTIVE`: Spring 프로파일 (`dev`, `prod`)
* `JWT_SECRET`: JWT 서명 키

## 로그/볼륨 설정 (선택)

* `/app/logs` 디렉토리에 로그 저장
* 예: `-v /your/log/dir:/app/logs`

## 헬스 체크

* URL: `http://localhost:8080/actuator/health`

## 참고

* Swagger: `http://localhost:8080/swagger-ui.html`
* GitHub: `https://github.com/your-org/user-service`