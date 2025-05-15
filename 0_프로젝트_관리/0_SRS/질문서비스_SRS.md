# Question Service SRS

1. **문서 개요 (Overview)**

    * 본 문서는 무물보 프로젝트의 **질문 서비스**(Question Service)의 요구사항을 정리한 문서입니다.
    * 관련 문서 또는 참고 링크
      * [질문 서비스 API Docs](https://github.com/A-OverFlow/mmb-docs/blob/main/0_%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8_%EA%B4%80%EB%A6%AC/1_%EA%B0%9C%EB%B0%9C/API_Docs/QUESTION_REST_API_Docs.md)
      * [질문 서비스 코드 저장소](https://github.com/A-OverFlow/mmb-question-service)

2. **서비스 개요**

    * 무물보의 질문 서비스는 질문 관련 기능을 제공하는 서비스입니다.
    * 주요 기능: 질문 등록, 조회, 수정, 삭제
    * 외부 또는 내부 사용자 (Client 등):
      * 외부 사용자: 서비스의 핵심 기능(등록, 조회, 수정, 삭제)을 사용하는 최종 사용자
      * 내부 사용자: N/A

3. **기능 명세 (Functional Requirements)**

    * 기능 목록 및 설명
      * 질문 등록
        * 질문 제목, 내용을 입력 후 등록을 요청하면 질문을 등록한다.
      * 질문 조회
        * 페이지별 질문 목록을 최신순으로 조회할 수 있다.
        * 단건 질문을 조회할 수 있다.
      * 질문 수정
        * 질문 제목, 내용, 상태를 수정 후 수정을 요청하면 질문을 수정한다.
      * 질문 삭제
        * 질문의 삭제를 요청하면 질문을 삭제한다.
    * 입력/출력 예시
      * 질문 생성 입력 예시
        ```json
        {
            "subject": "제목입니다.",
            "content": "내용입니다.",
            "authorId": 321,
            "authorName": "heejaykong",
        }
        ```
      * 질문 단건 조회 출력 예시
        ```json
        {
            "id": 1,
            "subject": "제목입니다.",
            "content": "내용입니다.",
            "author": {
                "id": 321,
                "name": "heejaykong"
            },
            "status": "NEW",
            "answers": [],
            "createdAt": "2025-05-06T23:26:04.436588",
            "editedAt": "2025-05-06T23:26:04.436588"
        }
        ```

4. **API 명세 요약**

    * 주요 API 목록
      * |HTTP Method|URL|비고|
        |------|---|---|
        |GET|https://mumulbo.com/api/v1/questions|질문 전체 조회|
        |GET|https://mumulbo.com/api/v1/questions/{questionId}|질문 단건 조회|
        |POST|https://mumulbo.com/api/v1/questions|질문 생성|
        |PUT|https://mumulbo.com/api/v1/questions|질문 수정|
        |DELETE|https://mumulbo.com/api/v1/questions/{questionId}|질문 삭제|

5. **도메인 모델 요약**

    * **Question (질문):** 사용자가 등록한 질문.
      * 주요 속성: 질문ID, 제목, 내용, 작성자ID, 작성자 이름, 질문 상태, 작성일, 수정일

6. **비기능 요구사항 (NFR)** - TBD

    * _보안_
    * _로깅_
    * _장애 대응 / 알림_
    * _배포 및 운영 고려사항 (환경변수, Helm 등 핵심만)_

7. **연동 서비스**

    * 의존하는 외부 서비스
      * 회원 서비스(Member Service)
      * 답변 서비스(Answer Service)
    * _Kafka topic / Redis key 등 메시지 통신 요약 - TBD_

8. **기타**

    * 용어 정의 (약어 등)
      * N/A
    * TODO 또는 추후 논의가 필요한 항목
      * 회원 서비스와 답변 서비스 간 데이터 통신 방법 논의 필요
      * 질문 삭제 정책 논의 필요(답변이 있는 질문은 삭제 불가 정책 등)
