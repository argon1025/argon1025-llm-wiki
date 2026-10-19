---
description: 원장·선생 업무 목록·캘린더·상세나 배정 후보를 조회할 때
---

# 원장·선생 업무 조회

## 적용 대상

- edu-fit-backend의 원장 업무 API(`/api/director/tasks` 이하)와 선생 업무 API(`/api/teacher/tasks` 이하), 배정 후보 조회(`GET /api/director/task-assignee-candidates`)

## 규칙

### 역할 제한

- 원장 업무 API(`/api/director/tasks` 이하)는 선생·학생 토큰을 403 `FORBIDDEN`으로 거부함
- 선생 업무 API(`/api/teacher/tasks` 이하)는 원장·학생 토큰을 403 `FORBIDDEN`으로 거부함
- 배정 후보 조회(`GET /api/director/task-assignee-candidates`)는 원장 토큰만 허용함

### 목록

- 원장·선생 업무 목록은 철회한 업무를 빼고 마감 기한·ID 오름차순의 `{ total, items }` 페이지로 반환함
- 목록 페이지 파라미터는 page 1~1000000(기본 1), limit 1~100(기본 20)임
- `hidden` 생략·false면 요청자가 숨긴 업무를 빼고, true면 숨긴 업무만 반환함
- `hidden`을 `status`와 함께 주면 두 조건을 모두 적용함

### 필터

- 원장 업무 목록은 `status`·`studentId`·`assigneeId`, 원장 업무 캘린더는 `studentId`·`assigneeId` 필터를 받음
- 원장 업무 목록·캘린더의 담당자 필터(`assigneeId`)는 현재 담당자 기준임
- 선생 업무 목록·캘린더는 `assigneeId`를 받지 않아 보내면 400임
- `studentId`·`assigneeId` 필터는 1~4294967295 정수만 받아 범위 밖 값은 빈 결과 대신 400 `INVALID_REQUEST`임

### 캘린더

- 원장·선생 업무 캘린더는 마감 기한 `from`~`to`(양끝 포함)의 철회되지 않은 업무를 반환함
- 캘린더 항목은 요청자가 숨긴 업무를 `isHidden` true로 표시함
- 원장 업무 캘린더는 미배정 업무도 반환함
- 캘린더는 마감 기한·ID 오름차순의 페이지 없는 배열이며 기간 상한이 없음
- `from`이 `to`보다 늦으면 400 `INVALID_REQUEST`(`from: from must be on or before to`)임

### 상세 응답

- 원장·선생 업무 상세는 학생의 현재 프로필(`student`의 학교·학교급·학년·졸업 여부)과 요청 시점의 학교급·학년·학기(`schoolLevel`·`grade`·`semester`)를 따로 반환함
- 상세의 첨부 파일에는 업로더(`uploaderId`)가 실림
- 선생 업무 상세(`GET /api/teacher/tasks/:taskId`)의 담당자(`assignee`)는 항상 요청한 선생 본인임(의도된 동작) — 현재 담당 업무만 조회되며 응답 형태 유지를 위해 남긴 필드임
- 미배정 업무는 담당자(`assignee`)가 null이고, 최초 배정 시각(`firstAssignedAt`)은 배정된 적 없는 업무만 null임

### 배정 후보

- 배정 후보 조회는 활성 원장·선생을 이름 오름차순(동명이면 ID 오름차순)으로 페이지 없이 반환함

## 함정

- 선생 업무 목록(`GET /api/teacher/tasks`)의 접수 상태 필터(`status=RECEIVED`)는 허용값이지만 항상 빈 결과임(정상) — 현재 담당 업무에는 접수 상태가 없음
