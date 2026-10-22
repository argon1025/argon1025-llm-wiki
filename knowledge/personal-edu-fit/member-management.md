---
description: 원장이 학생·선생 계정·담당 과목을 등록·조회·수정·비활성화하거나 학생 본인 프로필을 다룰 때
---

# 학생·선생 회원 관리

## 적용 대상

- edu-fit-backend의 원장 학생 API(/api/director/students), 원장 선생 API(/api/director/teachers), 학생 본인 프로필 API(/api/student/profile)
- 이 API를 호출하는 edu-fit-frontend의 회원 관리 화면과 원장 화면의 학생 선택지

## 규칙

### 학생 판정

- 없는 회원 ID·학생이 아닌 회원 ID·프로필이 없는 학생 계정은 학생 API에서 404 STUDENT_NOT_FOUND임 — 학생은 student_profile 행이 있는 회원임
- 다른 회원이 쓰는 로그인 ID와 대소문자·악센트만 다른 로그인 ID(student01이 있을 때 STUDENT01)로 학생을 등록·수정해도 409 LOGIN_ID_DUPLICATED임 — 로그인 ID 중복을 collation utf8mb4_0900_ai_ci로 판정함
- 본인 로그인 ID를 그대로 보내는 학생 수정은 성공함

### 학생 등록·수정

- 원장 학생 등록(POST /api/director/students)은 활성 학생 계정과 프로필을 한 번에 만들고 201로 응답함
- 학생 등록 시 학교급·학년은 현재 학년도 기준으로 저장함
- 원장 학생 수정(PATCH /api/director/students/:studentId)은 전달한 필드만 바꾸고 학교(school)·연락처(phone)는 null로 지움
- 학생 수정에서 loginId·name·schoolLevel·grade에 null을 보내면 400임
- 학생 수정에서 schoolLevel·grade 중 하나라도 오면 현재 학년도 기준으로 다시 지정함
- 학생 수정을 호출하는 edu-fit-frontend는 학교급·학년을 바꾸지 않았으면 schoolLevel·grade 두 값을 보내지 않음 — 하나라도 보내면 현재 학년도로 다시 지정됨
- grade 없이 schoolLevel만 오면 현재 표시 학년을 새 지정 학년으로 삼고, schoolLevel 없이 grade만 오면 기존 학교급을 유지함
- 학생 등록·수정 입력이 로그인 ID 4~50자, 이름 1~100자, 학교 100자 이하, 연락처 30자 이하, 학년 1~255 정수를 벗어나면 400 INVALID_REQUEST이며 길이는 코드 포인트 단위로 셈
- 학생 등록·수정은 학교급별 학년 범위(초 1~6, 중·고 1~3)를 검증하지 않음(의도된 동작)
- 학생 등록의 초기 비밀번호 최소 길이도 코드 포인트 단위로 셈
- edu-fit-frontend는 학생 등록·수정에서 이름(name) 앞뒤 공백을 제거한 뒤 필수 검증을 함 — edu-fit-backend는 이름 길이(1~100자)만 검증해 공백만 있는 이름도 받음

### 학생 상태·조회

- 원장 학생 비활성화(POST /api/director/students/:studentId/deactivate)·재활성화(POST /api/director/students/:studentId/activate)는 이미 같은 상태여도 200 data null로 성공함
- 원장 학생 비밀번호 재지정(PUT /api/director/students/:studentId/password)은 200 data null로 응답하며 기존 토큰은 유지됨
- 원장 학생 목록(GET /api/director/students)은 이름 오름차순에 동명이면 ID 오름차순이며 응답은 { total, items }임
- 학생 목록은 기본으로 활성 학생만 반환하고 includeInactive=true면 비활성도 포함함
- 활성 학생만 조회한 학생 목록의 total이 0이면 includeInactive=true&limit=1로 다시 조회해 등록된 학생이 없는지 가림 — total은 필터 조건에 맞는 건수라 활성 학생만 없는 경우와 구분되지 않음
- edu-fit-frontend는 원장 화면의 학생 선택지를 학생 목록(GET /api/director/students)의 이름(name) 부분 일치 검색 결과로 받음 — limit 상한이 100이고 전체 조회 옵션이 없음
- 학생 목록의 name은 대소문자 구분 없는 부분 일치이며 LIKE 특수문자는 글자 그대로 취급함
- 학생 목록 쿼리의 page가 1~1000000, limit이 1~100 정수가 아니거나 includeInactive가 'true'·'false'가 아니면 400 INVALID_REQUEST이며, 생략하면 page 1, limit 20임
- 학생 본인 프로필(GET /api/student/profile)은 id·이름·학교·학교급·표시 학년·졸업 여부만 반환하고 로그인 ID·연락처는 담지 않음
- 학생 본인 프로필 API에 원장·선생 토큰을 보내면 403 FORBIDDEN임

### 선생 회원

- 담당 과목이 없어도 역할이 선생(user.role = 'TEACHER')인 회원은 선생으로 처리함 — 선생은 학생의 student_profile 같은 별도 프로필 테이블이 없음
- 선생 본인은 별도 프로필 API 없이 GET /api/me를 씀
- 선생 담당 과목(teacher_subject)은 참고 정보이며 수업 설정·업무 배정을 제약하지 않음

### 선생 등록·수정

- 원장 선생 등록(POST /api/director/teachers)은 로그인 ID 4~50자, 이름 1~100자, 연락처 30자 이하(코드 포인트 기준)임
- 선생 등록의 초기 비밀번호와 재지정의 새 비밀번호 최소 길이도 코드 포인트 단위로 셈
- 선생 등록의 담당 과목 ID 배열(subjectIds)은 필수이며 빈 배열을 허용함
- 선생 등록의 subjectIds에 없거나 삭제된 과목이 하나라도 있으면 계정을 만들지 않고 404 SUBJECT_NOT_FOUND임
- 선생 등록·수정의 담당 과목 ID에 중복이 있어도 400이 아니라 한 번만 저장함
- 컬럼 범위를 넘는 담당 과목 ID(4294967296, 1e300)는 500이 아닌 404 SUBJECT_NOT_FOUND로 응답함
- 원장 선생 수정(PATCH /api/director/teachers/:teacherId)은 전달한 필드만 바꾸고 연락처(phone)만 null로 값을 지움
- 선생 수정에서 로그인 ID·이름·담당 과목 ID에 null을 보내면 400임
- 선생 수정에서 subjectIds를 보내면 담당 과목을 전체 교체하고, 생략하면 담당 과목을 유지함
- 전체 교체 시 이미 담당 중인 과목은 삭제된 과목이어도 유지하며 새로 추가하는 과목만 활성 과목이어야 함
- 새로 추가하는 과목이 404면 같이 보낸 이름·로그인 ID 변경도 저장되지 않음
- 선생 응답의 담당 과목(subjects)은 {id, name} 배열이며 삭제된 과목도 이름을 표시함
- subjects는 과목 이름 오름차순(같은 이름은 ID순)이며 담당 과목이 없으면 빈 배열임

### 선생 상태·조회

- 원장 선생 목록(GET /api/director/teachers)은 기본으로 활성 선생만 이름 오름차순(동명이면 ID 오름차순)으로 돌려주고 응답은 {total, items}임
- 선생 목록 응답 total은 필터 조건에 맞는 건수이며 전체 등록 선생 수가 아님
- 선생 목록은 includeInactive=true면 비활성도 포함하고, name은 부분 일치이며 LIKE 특수문자는 글자 그대로 취급함
- 선생 목록 쿼리는 page 1~1000000(기본 1), limit 1~100(기본 20), includeInactive는 'true'·'false'만 허용함
- 선생 비활성화(POST /api/director/teachers/:teacherId/deactivate)는 담당 과목 행도 보존하며, 비활성 선생의 로그인은 401 INVALID_CREDENTIALS, 기존 토큰 요청은 401 UNAUTHORIZED임
- 선생 비활성화·재활성화(POST /api/director/teachers/:teacherId/activate)는 이미 같은 상태여도 200 성공임
- 선생 비활성화 뒤 그 선생이 담당한 진행 중 업무는 원장이 재배정해야 다른 담당자로 넘어감 — 비활성화는 업무 담당자를 바꾸지 않음
- 선생 비밀번호 재지정(PUT /api/director/teachers/:teacherId/password)·비활성화·재활성화 응답 data는 null임
- 선생 비밀번호를 재지정해도 기존 토큰은 유지됨

### 선생 API 거부 응답

| 원장 선생 API 요청 조건 | 응답 |
|---|---|
| 없는 회원 ID나 선생이 아닌 회원(학생·원장) ID | 404 TEACHER_NOT_FOUND |
| 다른 회원이 쓰는 로그인 ID(대소문자 무시)로 등록·수정 | 409 LOGIN_ID_DUPLICATED |
| 숫자가 아닌 경로 ID | 400 |
| 선생·학생 토큰 | 403 FORBIDDEN |

## 코드값

원본: `MemberResponseCode` 전 4종

| 코드 | 뜻 |
|---|---|
| `STUDENT_NOT_FOUND` | 404 '존재하지 않는 학생입니다.' |
| `LOGIN_ID_DUPLICATED` | 409 '이미 사용 중인 로그인 ID입니다.' |
| `TEACHER_NOT_FOUND` | 404 '존재하지 않는 선생입니다.' |
| `ASSIGNEE_NOT_FOUND` | 404 '배정할 수 없는 담당자입니다.' |
