## 이번주 개인 로그
### API gateway 베이스 코드 작성
- 80번 포트로 진입하면 mmb-frontend 서비스로 보이도록 추가
  ![intro](../../9_images/cap1.png)
- 로컬 환경의 docker compose 로 다른 서비스들과 컨테이너 네트워크 생성 되는지 확인
```yml
version: '3.8'
services:
  eureka-server:
    build:
      context: ./eureka-server
    container_name: eureka-server
    ports:
      - "8761:8761"
    networks:
      - mmb-network
    environment:
      - EUREKA_SERVER=true

  api-gateway:
    build:
      context: ./mmb-apigateway
    container_name: api-gateway
    depends_on:
      - eureka-server  # Eureka 서버가 먼저 실행됨
    environment:
      - SPRING_PROFILES_ACTIVE=default
      - EUREKA_SERVER_URI=http://eureka-server:8761/eureka/
    ports:
      - "80:80"
    networks:
      - mmb-network

  frontend-service:
    image: mmb-frontend
    build:
      context: ./mmb-frontend
    container_name: frontend-service
    ports:
      - "3000:80"
    networks:
      - mmb-network

#  member-service:
#    build:
#      context: ./member-service
#    container_name: member-service
#    ports:
#      - "8082:8082"
#    networks:
#      - mmb-network
#    environment:
#      - SPRING_PROFILES_ACTIVE=default


networks:
  mmb-network:
    driver: bridge

```  

  - eureka 클라이언트 등록 및 멤버 서비스도 eureka 클라이언트로 등록 되었을 때 api gw에서 연결 가능한지 확인 (멤버 서비스 부분은 코드 반영X)
    ![intro](../../9_images/cap2.png)
  - deploy.yml 작성 후 테스트용 도커 허브에 이미지 Push까지 해보기 -> 실패 !!!
    - 로컬에서 해당 도커 허브에 접근이 되지만 github action 에서 도커 허브로 접근이 안됨. yml 파일이나 secrets 설정 문제일가능성이 커서.. 인프라 파트원들과 논의 예쩡


## 다음주 todo 
- config server 적용 여부 판단 
- 회원/질문 서비스로 포트 라우팅 및 docker compose 로 확인 
- docker-compose 용으로 deploy.yml 수정 (apigateway.yml 은 이미 docker-compose용으로 개발함)
