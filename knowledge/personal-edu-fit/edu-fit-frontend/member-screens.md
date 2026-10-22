---
description: 학생·선생 관리 목록·수정 모달이나 담당 학생 목록·학년 입력을 고칠 때
---

# 회원 관리 화면

## 규칙

- 학생 등록·수정 폼(useStudentForm)은 로그인 ID를 trim해 보냄 — 로그인 폼(LoginForm)이 ID 앞뒤 공백을 지워 보내므로 공백이 섞인 ID로 만든 학생은 로그인할 수 없음
- 학생 수정 모달(useStudentForm)은 졸업 표시 학생의 선택지 밖 학년(예: 중등 4)도 학교급·학년을 건드리지 않으면 유효로 두고, 둘 중 하나라도 바꾸면 학년을 다시 고르게 함 — 다른 필드만 고쳐 저장할 수 있게 함
- 담당 학생 학년 수정(useGradeForm)은 졸업으로 표시되는 학생도 다음 학교급을 제안하지 않고 백엔드의 학교급·학년을 그대로 채우며, 표시 학년이 선택지 범위를 넘으면 학년을 미선택으로 두어 저장이 비활성임
- 담당 학생 목록(TeacherStudents)은 URL의 page가 쪽 수를 넘으면 effect에서 마지막 쪽으로 보정하며 렌더 중에는 URL을 바꾸지 않음
- 학생·선생 수정 모달은 404(STUDENT_NOT_FOUND·TEACHER_NOT_FOUND)면 모달을 닫고 목록·존재 조회 키(studentKeys.lists()·studentKeys.hasAny() 등)만 무효화하며 같은 토스트가 떠 있으면 다시 띄우지 않음
- 학생·선생 관리 목록의 빈 상태는 활성만·검색어 없음·0건일 때만 includeInactive=true&limit=1 존재 조회(studentQueries.hasAny·teacherQueries.hasAny)로 등록 없음과 활성 없음을 가름
- 선생 등록·수정 모달의 담당 과목 선택지(SubjectOption)는 활성 과목에 현재 담당 중인 삭제된 과목(deleted)을 '삭제됨'으로 더하며, 삭제된 과목은 해제만 가능하고 해제 뒤 다시 선택할 수 없음
- 선생 폼(useTeacherForm)이 보내는 subjectIds는 현재 선택지에 있는 ID만 담음 — 모달을 연 사이 삭제돼 404 뒤 선택지에서 빠진 과목이 해제 불가한 채 계속 전송되는 것을 막음

## 함정

- 수정 모달의 404 처리가 studentKeys.all처럼 상세 조회 키까지 무효화하면 모달이 닫히는 100ms 동안 남은 상세 조회가 다시 404를 받아 같은 처리를 반복해 상세 요청이 19회 나감
- AccountSection의 비밀번호 초기화 폼은 제출 핸들러에서 stopPropagation을 호출함 — 포털로 그려져도 React 합성 제출 이벤트가 React 트리를 따라 바깥 회원 수정 form의 onSubmit까지 번짐

## 결정

- 담당 학생 학년 입력은 학생 관리의 학교급 세그먼트+학년 칩 입력(GradeFields)을 쓰지 않고 학교급·학년 Select 2개(GradeSelects)로 둠 — 인라인 편집 행에 맞춘 형태이며 회원 화면 사이 학년 입력 형태가 다른 것을 감수함
