작성기한: 2025-05-29

# 12주차 - 질문 생성 기능 리팩토링(X-User-Id 등).md

## 지난 주 목표
1. 작성자 id를 요청 body가 아닌 커스텀 헤더 X-User-Id에서 받도록 리팩토링 [(#37)](https://github.com/A-OverFlow/mmb-question-service/issues/37)
2. 질문 생성 요청 변경에 따른 질문서비스 API 문서 수정 [(#57)](https://github.com/A-OverFlow/mmb-docs/issues/57)
3. docker 설정파일 수정 반영(feat. 심선임님 가이드) [(#38)](https://github.com/A-OverFlow/mmb-question-service/issues/38)
4. 내가 작성한 질문 목록 조회 API 항목 작성 [(#58)](https://github.com/A-OverFlow/mmb-docs/issues/58)
5. 내가 작성한 질문 목록 조회 API 구현 [(#40)](https://github.com/A-OverFlow/mmb-question-service/issues/40)

## 완료한 작업
1. 작성자 id를 요청 body가 아닌 커스텀 헤더 X-User-Id에서 받도록 리팩토링 [(#37)](https://github.com/A-OverFlow/mmb-question-service/issues/37) ✅
    * [PR 링크](https://github.com/A-OverFlow/mmb-question-service/pull/39)
2. 질문 생성 요청 변경에 따른 질문서비스 API 문서 수정 [(#57)](https://github.com/A-OverFlow/mmb-docs/issues/57) ✅
    * [문서 링크](https://github.com/A-OverFlow/mmb-docs/commit/949d7660c0ec7e8604a7f5a79d1ece2ac1cfdfaf)

## 진행 중인 작업
3. docker 설정파일 수정 반영(feat. 심선임님 가이드) [(#38)](https://github.com/A-OverFlow/mmb-question-service/issues/38)
4. 내가 작성한 질문 목록 조회 API 항목 작성 [(#58)](https://github.com/A-OverFlow/mmb-docs/issues/58)
5. 내가 작성한 질문 목록 조회 API 구현 [(#40)](https://github.com/A-OverFlow/mmb-question-service/issues/40)

## 배운 점
* 아직 없음
  * 서비스에서 서비스로 통신 요청할 때(질문서비스 -> 멤버서비스) 일단 OpenFeign을 사용해보려고 하고는 있음
  * 참고 자료
    * [Spring 공식 문서 - Spring Cloud OpenFeign](https://spring.io/projects/spring-cloud-openfeign)
    * [우아한형제들 기술블로그 - 우아한 feign 적용기](https://techblog.woowahan.com/2630/)

## 개선할 점
* N/A
 
## 기타 공유 사항
* docker compose 파일 고치다가 질문이 생겼습니다. 이따 모각코때 선임님께 Help!

## 다음 주 계획
* 서비스간 무사 통신 기원

끝.
