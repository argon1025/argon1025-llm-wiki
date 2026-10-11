---
description: 수업 설정·시간표 수업을 저장·조회·삭제하거나 시수를 대조할 때
---

# 수업 설정과 시간표

## 적용 대상

- 수업 설정과 수업을 저장·삭제하는 edu-fit-backend, 수업 설정으로 시수를 계산하는 edu-fit-frontend

## 규칙

- 활성 수업 설정은 삭제되지 않은(deletedAt null) 설정이며 기간 종료일(endDate)과 무관하고, 선생 담당 학생 판정과 목록 조회가 이 기준을 씀
- 원장 수업 설정 생성(POST /api/director/courses)은 시작일·종료일 순서 → 학생 → 과목 → 담당 선생 순으로 검증함
- 생성에서 시작일(startDate)이 종료일(endDate)보다 늦으면 400 INVALID_REQUEST이고 같은 날은 허용함
- 생성에서 학생 프로필 있는 회원이 아니면 404 STUDENT_NOT_FOUND, 선생이 아니면 404 TEACHER_NOT_FOUND, 없거나 삭제된 과목은 404 SUBJECT_NOT_FOUND이며 비활성 학생·선생은 허용함
- 수업 설정 수정은 주 횟수·회당 분·기간·기본 강의실·세부사항만 허용하고 학생·과목·담당 선생은 고정함 — 선생 교체는 새 설정 생성으로 처리해 기존 수업·기록의 담당 이력을 보존함
- 수업 설정 수정(PATCH /api/director/courses/:courseId)은 전달된 필드만 바꾸고, 학생·과목·담당 선생(studentId·subjectId·teacherId)을 보내면 미정의 필드로 400 INVALID_REQUEST임
- 수정에서 startDate·endDate·defaultRoom·details는 null로 지울 수 있고 weeklyCount·minutesPerLesson의 null은 400 INVALID_REQUEST임
- 수정은 기존 값에 요청 값을 반영한 뒤 시작일이 종료일보다 늦으면 전용 코드 없이 400 INVALID_REQUEST로 거부하고 아무것도 바꾸지 않으며, 둘 중 하나라도 null이면 비교하지 않음
- 수업 설정 삭제는 설정과 연결 수업을 함께 소프트 삭제해 시간표에서만 빼고 수업 기록은 보존함
- 수업 설정 삭제 성공은 200 data null이고 없거나 삭제된 수업 설정의 상세·수정·삭제와 재삭제는 404 COURSE_NOT_FOUND(존재하지 않는 수업 설정입니다.)임
- 원장 수업 설정 목록(GET /api/director/courses)은 삭제되지 않은 설정을 학생 이름 → 과목 이름 → ID 오름차순으로 page(1~1000000, 기본 1)·limit(1~100, 기본 20) { total, items }로 돌려줌
- 원장 목록은 studentId·teacherId·subjectId 필터를 함께 적용하고 종료일이 지난 설정과 삭제된 과목 이름도 포함함
- 학생의 수업 설정 조회(GET /api/student/courses)는 본인이 학생인 삭제되지 않은 설정을 과목 이름 → ID 순으로 페이지 없이 반환하고 종료일이 지난 설정을 포함함
- 학생 조회는 학생 정보와 세부사항(details)을 싣지 않으며 설정이 없거나 학생 프로필이 없는 계정은 빈 배열임
- 선생의 수업 설정 조회(GET /api/teacher/courses)는 본인이 담당 선생인 삭제되지 않은 설정을 학생 이름 → 과목 이름 → ID 순으로 페이지 없이 반환하고 종료일이 지난 설정을 포함하며, page 쿼리는 400임
- 선생 조회는 담당 선생과 세부사항(details)을 싣지 않으며 studentId 필터가 담당하지 않는 학생이면 빈 배열임
- 수업 설정 API는 원장 경로(/api/director/courses)의 모든 동작이 원장 전용이고 조회는 학생·선생 경로가 각각 본인 소속 설정만 다루며, 다른 역할 토큰은 403 FORBIDDEN, 토큰이 없으면 401 UNAUTHORIZED임
- 수업 설정 입력 상한은 weeklyCount·minutesPerLesson 1~65535 정수, startDate·endDate 1000~9999년의 실재하는 YYYY-MM-DD임
- 수업 설정 입력 상한은 기본 강의실(defaultRoom) 코드 포인트 0~100자, 세부사항(details) UTF-8 0~65535 bytes임
- 입력 상한을 어기면 400 INVALID_REQUEST로 거부하고 저장하지 않음
- 수업(Lesson)의 room 컬럼과 요청 검증은 코드 포인트 100자 이상을 받아야 함 — room을 생략하면 수업 설정의 defaultRoom을 복사해 저장하므로 더 짧으면 Data too long 500이 됨
- 수업 개별 삭제도 소프트 삭제임
- 회당 수업 시간은 분 단위 양의 정수이며 30분 등 배수를 강제하지 않음
- 시수 대조는 백엔드 API 없이 edu-fit-frontend가 수업 설정의 주 횟수 × 회당 분과 기간 수업 조회 결과로 현지 주 기준 계산함
- 수업(Lesson)은 수업 설정(Course) 하나에 속한 실제 날짜·시간의 개별 수업이며 학생·선생·과목은 소속 수업 설정에서 정해지고 수업 자체에는 시작·종료 시각(startAt·endAt)과 강의실(room)만 있음
- 기간 수업 조회(GET /api/{director|teacher|student}/lessons)는 시작 시각(startAt)이 [from, to)인 수업만 시작 시각 → ID 오름차순의 페이지 없는 배열로 반환함
- 기간 수업 조회는 from 전에 시작해 구간과 겹치기만 하는 수업을 뺌 — 주간·월간 경계 수업이 두 구간에 중복되지 않음
- 기간 수업 조회에서 from ≥ to이거나 to가 from + 62일(UTC 62×24시간)을 넘으면 400 INVALID_REQUEST이고 62일 정각은 허용함
- 수업 조회·생성·수정의 시각 입력(from·to·startAt·endAt)은 Z 접미 UTC ISO 8601(YYYY-MM-DDTHH:mm:ss[.SSS]Z)이며 1000~9999년의 실재하는 날짜·시각만 받음
- 시각 입력에 +09:00 같은 오프셋·2026-02-30·24:00·60초가 있으면 400 INVALID_REQUEST임
- 주간·월간 경계는 edu-fit-frontend가 현지 시각으로 계산해 UTC 구간으로 보냄
- 수업 생성·수정의 startAt·endAt은 초 미만을 버려 저장·응답하며 밀리초 입력은 거부하지 않음
- 수업 생성·수정에서 초 미만을 버린 뒤 종료가 시작보다 늦지 않으면 400 LESSON_TIME_INVALID(종료 시각은 시작 시각보다 늦어야 합니다.)이고, 수정은 기존 값에 요청 값을 반영한 뒤 비교함
- 수업 생성(POST /api/{director|teacher}/lessons)은 삭제되지 않은 수업 설정(courseId)에만 가능해 없거나 삭제된 설정은 404 COURSE_NOT_FOUND임
- 수업 생성에서 room을 생략하면 설정의 기본 강의실(defaultRoom, 없으면 null)을 복사하고 null을 보내면 강의실 없이, 빈 문자열은 그대로 저장하며 코드 포인트 100자를 넘으면 400임
- 수업 수정(PATCH /api/{director|teacher}/lessons/:lessonId)은 전달한 필드만 바꾸고 room만 null로 지울 수 있으며 startAt·endAt null은 400임
- 수업 수정에서 소속 수업 설정(courseId) 등 미정의 필드는 400 INVALID_REQUEST이고, 빈 본문은 아무것도 바꾸지 않고 현재 항목을 200으로 반환함
- 수업 삭제(DELETE /api/{director|teacher}/lessons/:lessonId)는 소프트 삭제로 시간표에서만 빼고 수업 기록은 보존하며 성공 data는 null임
- 없거나 이미 삭제된 수업·삭제된 수업 설정에 속한 수업은 조회에서 빠지고 수정·삭제가 구분 없이 404 LESSON_NOT_FOUND(존재하지 않는 수업입니다.)임
- 수업 설정 삭제(DELETE /api/director/courses/:courseId)는 설정과 그 설정의 삭제되지 않은 수업에 같은 삭제 시각을 한 트랜잭션으로 기록해 세 역할의 시간표 조회에서 뺌
- 수업 설정 삭제는 먼저 개별 삭제된 수업의 삭제 시각과 다른 설정의 수업을 바꾸지 않음

| 수업 API 역할 | 대상 수업 | 허용 동작 |
|---|---|---|
| 원장 | 모든 수업 | 기간 조회(studentId·teacherId 필터)·생성·수정·삭제 |
| 선생 | 소속 수업 설정의 teacherId가 본인인 수업 | 기간 조회(studentId 필터)·생성·수정·삭제 |
| 학생 | 본인 수업 | 기간 조회 |

- 학생 수업 조회(GET /api/student/lessons)는 로그인 학생이 소속 수업 설정의 학생인 수업만 { id, subject, teacher, startAt, endAt, room }으로 반환함
- 학생 수업 조회는 수업 기록(latestNote 키 포함)·courseId·학생 필드를 싣지 않으며 프로필 없는 학생 계정은 빈 배열 200임

## 함정

- edu-fit-backend는 같은 (학생, 과목, 담당 선생) 활성 수업 설정 중복과 주 횟수 1~7 범위를 막지 않고 edu-fit-frontend가 판단함
- edu-fit-backend는 수업 저장 시 종료 > 시작만 검증하고 운영 시간(09:00~22:00)·30분 단위는 검증하지 않음(의도된 동작) — 운영 시간이 확장될 수 있어 시각을 자유롭게 저장하게 둠
- edu-fit-backend는 수업 저장 시 학생 이중 배정을 검증하지 않음(의도된 동작)
- edu-fit-backend는 수업 생성·수정에서 수업 설정 기간(startDate~endDate) 밖 시각과 비활성 학생·선생도 확인하지 않고 종료 > 시작과 설정 존재·역할 소유권만 검증함
- 탐구 업무로만 담당하는 학생의 수업은 선생 수업 API 대상이 아님 — 선생 수업 범위는 담당 학생 판정(teacher-scope)을 쓰지 않음
- 선생이 타 선생 설정 courseId로 수업을 만들면 404 COURSE_NOT_FOUND이고, 타 선생 설정의 수업을 수정·삭제하면 403이 아닌 404 LESSON_NOT_FOUND임
- 선생의 수업 조회에서 담당하지 않는 학생 studentId 필터는 빈 배열임
- 수업 응답의 과목 이름(subject.name)은 삭제된 과목도 그대로 표시함
