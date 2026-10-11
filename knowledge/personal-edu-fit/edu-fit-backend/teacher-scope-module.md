---
description: 선생 서비스에서 teacher-scope로 담당 학생 범위를 판정할 때
---

# 선생 담당 범위 모듈

## 규칙

- 선생 서비스가 TeacherScopeExternalService를 호출하면 teacherId로 @Roles('TEACHER') 컨트롤러의 로그인 회원 ID만 넘김 — 이 서비스는 teacherId의 역할과 학생 프로필 존재를 확인하지 않음

## 함정

- isInScope는 학생 한 명을 판정할 때도 선생의 담당 학생 ID 전체를 읽고, 선생 담당 학생 목록(getStudents)·선생 리포트 목록(getReports)은 그 ID 전체를 IN 조건으로 넘김 — 선생당 담당 학생 수백 명 이하를 전제로 한 의도적 단순화
- 선생당 담당 학생이 수천 명 규모가 되면 담당 판정을 학생 ID 조건 count External 메서드나 SQL 서브쿼리 조건으로 바꿔야 함
