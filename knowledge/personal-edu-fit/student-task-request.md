---
description: 학생 탐구 요청을 등록·수정·철회·숨김하거나 목록·캘린더로 조회할 때
---

# 학생 탐구 요청

## 적용 대상

- 학생 토큰으로 호출하는 학생 탐구 요청 API(/api/student/tasks)를 구현하는 edu-fit-backend와 이를 호출하는 edu-fit-frontend
- 원장·선생용 요청 API는 제외

## 규칙

### 등록

- 학생 탐구 요청 등록(POST /api/student/tasks)은 201로 접수(RECEIVED)·담당자 미배정(assignee null) 상세를 돌려줌
- 등록에서 학교급(schoolLevel)·학년(grade)·학기(semester)·마감 기한(dueDate)·요청 서비스(services)·과목 및 유형(subjectAndType)·두 첨부 ID 배열(noticeFileIds, workFileIds)은 필수임
- 학년은 1~255 정수, 학기는 1 또는 2, 마감 기한은 실제 있는 YYYY-MM-DD 날짜임
- 과목 및 유형(subjectAndType)은 1~255 코드 포인트이며 공백만으로는 불가함
- 전달 내용(message)은 UTF-8 65,535 bytes 이하임
- 등록 요청에 정의되지 않은 필드가 있으면 400 INVALID_REQUEST임
- 탐구 축 미지정은 axisId null임
- 없거나 삭제된 탐구 축(axisId)은 404 INQUIRY_AXIS_NOT_FOUND임(등록·수정 공통)

### 첨부

- 두 첨부 ID 배열(noticeFileIds·workFileIds)은 배열 순서가 첨부 순서이며 분류 안 중복 ID는 한 번만 연결됨
- 다른 회원이 업로드한 파일을 새로 연결하면 400 FILE_NOT_OWNED로 거부하고 요청을 저장하지 않음
- 없는 파일 ID는 404 FILE_NOT_FOUND임(등록·수정 공통)

### 조회

- 상세 응답의 첨부는 noticeFiles·workFiles로 나뉘어 연결 순서대로 오고 각 파일은 서명 다운로드 경로(downloadUrl)를 가짐
- 상세 응답의 담당자(assignee)는 { id, name } 또는 미배정 null임
- 등록 뒤 탐구 축이 삭제되면 상세는 axisId를 null(미분류)로 반환함
- 목록(GET /api/student/tasks)은 본인의 철회되지 않은 요청을 마감 기한·ID 오름차순으로 page(1~1000000, 기본 1)·limit(1~100, 기본 20) 페이지 { total, items }로 돌려주며 status 필터는 선택임
- 목록의 hidden 생략·false는 본인이 숨긴 요청을 빼고 true는 숨긴 요청만 반환함
- hidden을 status와 함께 주면 두 조건을 모두 적용하고, status를 생략한 hidden=true는 모든 상태의 숨긴 요청임
- 캘린더(GET /api/student/tasks/calendar)는 마감 기한이 from~to(양끝 포함)인 본인 요청을 숨김 건까지 포함해 마감 기한·ID 오름차순 배열로 반환함
- 캘린더에서 from이 to보다 늦거나 날짜 형식이 틀리면 400 INVALID_REQUEST임

### 수정

- 수정(PATCH /api/student/tasks/:taskId)은 전달한 필드만 바꾸고 빈 본문은 아무것도 바꾸지 않음
- axisId·message는 null로 값을 지우고 나머지 필드의 null은 400 INVALID_REQUEST임
- 첨부 분류(noticeFileIds·workFileIds)를 전달하면 그 분류의 연결 전체를 배열 순서로 교체하고(빈 배열은 모두 해제) 전달하지 않은 분류는 유지함
- 이 업무에 이미 연결된 파일(분류 무관, 원장 대리 수정으로 연결된 파일 포함)은 업로더 확인에서 빠짐
- 검증에 실패하면 다른 필드도 반영되지 않음

### 철회·숨김

- 철회(DELETE /api/student/tasks/:taskId)는 deletedAt을 채우는 소프트 삭제로 200 data null을 반환하고 상세·목록·캘린더에서 빠지며 첨부 연결·숨김 행은 남음
- 숨김(PUT /api/student/tasks/:taskId/hidden)과 복원(DELETE 같은 경로)은 멱등이라 반복해도 200 data null임
- 숨김은 사용자별이라 다른 회원(원장 등)의 숨김은 학생의 isHidden에 반영되지 않고 학생의 복원도 다른 회원 숨김을 지우지 않음

### 접근

- 학생 요청 API는 학생 토큰만 허용해 원장·선생 토큰은 403 FORBIDDEN, 토큰 없음은 401 UNAUTHORIZED임
- 상세·수정·철회·숨김·복원에서 없는 업무·다른 학생의 업무·철회한 업무는 구분 없이 404 TASK_NOT_FOUND임

### 프론트 판단 범위

- edu-fit-backend는 빈 services 배열도 받은 그대로 저장함 — 요청 서비스 1개 이상 선택은 edu-fit-frontend가 판단함
- edu-fit-backend는 수정·철회에서 업무 상태를 보지 않고 본인 요청 여부만 확인함 — 접수 상태 한정 수정·철회는 edu-fit-frontend가 판단함

## 코드값

### 요청 서비스(services 배열 원소)

원본: `TaskService` 전 5종

| 코드 | 뜻 |
|---|---|
| `REPORT_WRITING` | 보고서 작성 |
| `FEEDBACK` | 피드백 요청 |
| `TOPIC_RECOMMENDATION` | 탐구주제 추천 |
| `RECORD_LINKAGE` | 생활기록부 연계 |
| `OTHER` | 기타 |

### 요청 첨부 분류

원본: `TaskFileCategory` 전 2종

| 코드 | 뜻 | 요청 필드 | 응답 필드 |
|---|---|---|---|
| `NOTICE` | 학교 안내문·수행평가 자료 | `noticeFileIds` | `noticeFiles` |
| `WORK` | 학생 작성 자료 | `workFileIds` | `workFiles` |
