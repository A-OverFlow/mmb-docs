# Member Service SRS

## 1. 문서 개요 (Overview)

이 문서는 `무물보 회원 서비스`에 대한 명세를 정의합니다.

## 2. 서비스 개요

`mmb-member-service(이하 회원 서비스)`는 회원 정보를 관리하는 서비스입니다.

3. **기능 명세 (Functional Requirements)**

    * 기능 목록 및 설명 (간단한 표 또는 리스트)
    * 입력/출력 예시 (간단한 JSON 예시로)

4. **API 명세 요약**

    * 주요 API 목록 (HTTP Method, URI, 설명만 간단히)
    * 인증 방식 (JWT, OAuth 등 간단 표기)

## 5. 도메인 모델 요약

### 주요 Entity

`Member`: 구글 OAuth 인증을 받은 회원 정보를 저장합니다.  
`Profile`: 회원의 추가 정보를 저장합니다.

### ERD Diagram

![member_erd.png](../../9_images/member_erd.png)

6. **비기능 요구사항 (NFR)**

    * 보안
    * 로깅
    * 장애 대응 / 알림
    * 배포 및 운영 고려사항 (환경변수, Helm 등 핵심만)

## 7. 연동 서비스

    * 의존하는 외부 서비스
    * Kafka topic / Redis key 등 메시지 통신 요약

8. **기타**

    * 용어 정의 (약어 등)
    * TODO 또는 추후 논의가 필요한 항목
