---
description: 도메인 모듈 경계·External 의존·DTO·interface 재사용 범위를 정할 때
---

# 도메인 모듈 경계

## 적용 대상

- edu-fit-backend의 도메인 모듈과 그 하위 controller·service·dto·interface 구성

## 규칙

- 도메인 경계는 인증, 회원, 수업 과목, 수업(수업 설정·수업·수업 기록), 리포트, 탐구 축, 컨설팅 업무, 탐구 분석, 성취도, 파일의 10개로 나눔
- FK는 같은 도메인 하위 테이블까지만 걸고 타 도메인 테이블은 ID로만 참조함 — 도메인 간 결합 방지
- 타 도메인 테이블의 표시 값은 애플리케이션에서 별도 조회해 조합함
- 도메인별 External 모듈은 External 서비스만 provide·export함
- External 서비스는 역할 서비스에 위임하지 않고 엔티티 매니저(EntityManager)로 직접 조회하며 다른 External 모듈을 import하지 않음 — 순환 의존 방지
- controller·service·dto·interface 아래에 역할(원장·선생·학생) 하위 폴더를 두고, 역할별 컨트롤러·서비스는 서로 독립임
- 요청·응답 DTO와 서비스 Option·Result interface는 도메인·역할·엔드포인트·계층(컨트롤러·서비스)마다 따로 선언하고, 기능이 완전히 같아도 다른 곳의 것을 재사용하지 않음 — 계층마다 바뀌는 원인이 다름
- 다른 엔드포인트 DTO를 `extends`·`Partial<>`·타입 별칭·공용 베이스 클래스로 가져오지 않고 목록 요청의 page·limit까지 엔드포인트마다 필드를 직접 선언함
- 서비스는 컨트롤러 DTO를 import하지 않으며, 컨트롤러가 요청 DTO 값을 서비스 Option으로 옮겨 담고 서비스 Result를 응답 DTO로 변환함
- 타 도메인 External 서비스가 돌려준 Result는 호출한 서비스 안에서 자기 interface로 변환해 쓰고 그대로 반환하거나 재export하지 않음
- enum(유니언 타입 포함)만 재사용을 허용하며, 여러 도메인이 쓰는 값은 공용 타입 모듈에, 도메인 고유 값은 소유 도메인에 둠
- 역할별 DTO 클래스명에는 역할 접두어를 붙임 — Swagger가 클래스명을 스키마 키로 써서 역할 간 동명 DTO가 덮어써짐
- 선생 담당 학생 판정은 수업 설정·업무 External 모듈만 import하는 전용 모듈 하나가 제공하고, 회원·리포트·탐구 분석·성취도의 선생 서비스가 씀 — 판정 중복과 순환 의존을 함께 피함
