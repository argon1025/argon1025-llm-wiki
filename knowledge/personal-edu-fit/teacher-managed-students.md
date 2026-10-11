---
description: 선생이 담당 학생 목록·상세를 조회하거나 학년을 수정할 때
---

# 선생 담당 학생

## 적용 대상

- edu-fit-backend의 선생 담당 학생 API — 원장·학생용 API는 제외

## 규칙

- 선생 담당 학생 API는 목록(GET /api/teacher/students)·상세(GET /api/teacher/students/:studentId)·학년 수정(PATCH /api/teacher/students/:studentId/grade) 3종임

### 거부 응답

| 선생 담당 학생 API 요청 조건 | 응답 |
|---|---|
| 원장·학생 토큰 | 403 FORBIDDEN |
| 담당 범위 밖 학생·없는 회원 ID·학생 프로필(student_profile)이 없는 회원 | 404 STUDENT_NOT_FOUND — 범위 밖 학생의 존재를 드러내지 않음 |
| 숫자가 아닌 학생 ID | 400 INVALID_REQUEST |

### 목록·상세

- 목록은 이름 오름차순(동명이면 ID 오름차순)의 { total, items }로 응답함
- 목록의 page는 1~1000000(기본 1), limit은 1~100(기본 20)이며 범위 밖이면 400 INVALID_REQUEST임
- 목록의 name 필터는 부분 일치이고 LIKE 특수 문자(\ % _)는 글자 그대로 비교함
- 목록·상세 응답의 relationTypes는 COURSE·CONSULTING 중 1개 이상을 COURSE → CONSULTING 순으로 나열함
- 상세 응답은 목록 항목 필드에 연락처(phone)를 더하며 연락처가 없으면 null임

### 학년 수정

- 학년 수정은 schoolLevel·grade를 모두 필수로 받으며 누락·null·허용 외 학교급·미정의 필드는 400 INVALID_REQUEST임
- 학년 수정의 grade는 1~255 정수이며 학교급별 학년 범위는 검증하지 않음
- 학년을 저장하면 지정 학년도(gradeBaseYear)가 현재 학년도로 갱신되어 응답 표시 학년이 요청 학년과 같음

## 함정

- 선생 담당 학생 목록·상세는 비활성 학생을 포함하지만 응답에 status 필드가 없어 선생 화면이 비활성 학생을 구분할 수 없음
