---
description: 목록 조회에서 검색어 LIKE·서브쿼리 조건·이름·학교급 정렬을 구현할 때
---

# 목록 조회 구현

## 규칙

- 원장 학생·원장 선생 서비스가 함께 쓰는 LIKE 검색어 이스케이프 함수(escapeLike)는 member 도메인 루트에 둠 — 역할별 서비스끼리 import하지 않음
- MikroORM 7에서 em.find 조건에 서브쿼리를 넣을 때는 em.createQueryBuilder(Entity).select(...).where(...).getNativeQuery().toRaw() 결과를 $in·$nin 값으로 씀
- course 도메인의 이름 정렬(compareName)에 'ko' 로캘을 쓰지 않음 — 'ko'는 한글을 라틴 문자보다 앞에 두어 MySQL utf8mb4_0900_ai_ci 이름순과 달라짐
- 탐구 이력 목록의 학교급 정렬(compareInquiryRecords)은 SQL이 아닌 애플리케이션에서 함 — 학교급 문자열 사전순(ELEMENTARY < HIGH < MIDDLE)이 학교급 순서와 다름

## 결정

- 원장 수업 설정 목록(getCourses)은 이름 정렬 External 조회나 정렬용 이름 컬럼 저장 없이 조건에 맞는 설정 전체를 메모리에서 정렬해 페이지를 자름(의도적 단순화) — 설정 수천 건 이하 규모를 전제로 하며 수만 건이 되면 둘 중 하나로 바꿔야 함
- 최신 수업 기록 조회(findLatestLessonNotes)는 윈도 함수 원시 SQL 대신 em.find 두 번(식별 컬럼 fields: ['lesson'] 조회 → 고른 기록만 refresh: true 재조회)으로 수업별 최신 기록을 고름
