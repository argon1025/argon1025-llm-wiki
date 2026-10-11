---
description: 컨설팅 업무 배정·상태 전이·산출물 완료·히스토리·숨김·철회를 다룰 때
---

# 컨설팅 업무 처리 흐름

## 적용 대상

- 업무 배정·상태 변경·히스토리를 검증하고 저장하는 edu-fit-backend, 배정·상태 변경·산출물 저장 API를 호출하는 edu-fit-frontend

## 규칙

- 업무 상태는 접수·진행 중·완료 3단계이며 접수는 담당자 미배정과 동치임
- 미배정 업무의 배정 취소·상태 변경은 edu-fit-backend가 409 TASK_NOT_ASSIGNED로 거부함 — 접수는 미배정과 동치라 대상 상태가 없음
- 배정 취소·재배정으로 빠진 이전 담당자는 업무 쓰기를 할 수 없고 현재 담당자만 가능함
- 배정·재배정·배정 취소·상태 변경·신청 내용 대리 수정 이력은 시스템 댓글이 아니라 별도 히스토리 테이블(task_history)에 이벤트 타입과 한국어 문구로 남기는 단순 감사 기록이며 원장·선생이 조회함
- 학생의 요청 철회는 업무를 소프트 삭제하며 산출물·댓글은 보존함
- 산출물은 별도 공유 상태 없이 저장 즉시 학생에게 노출됨
- 원장의 내 업무는 별도 API 없이 원장 업무 목록의 담당자 필터로 조회함
- 숨김은 사용자별 개인 설정이며 목록에만 적용하고 캘린더에는 적용하지 않으며 업무 상태가 아님

### 배정

- 배정 대상은 활성 원장·선생(원장 본인 포함)뿐이며 비활성 선생·학생·없는 회원·컬럼 범위를 넘는 ID는 404 ASSIGNEE_NOT_FOUND로 거부하고 업무를 바꾸지 않음
- 업무 배정 API(PUT /api/director/tasks/:taskId/assignee)는 업무 존재(404 TASK_NOT_FOUND)를 담당자 확인(404 ASSIGNEE_NOT_FOUND)보다 먼저 판정함
- 재배정은 담당자만 바꾸고 상태·산출물·댓글·최초 배정 시각(firstAssignedAt)을 유지하며 REASSIGNED 히스토리를 남김
- 현재 담당자와 같은 담당자로 재배정을 요청하면 상태·히스토리·배정 이력을 바꾸지 않고 현재 값으로 200을 응답함 — 재배정 후보에서 현재 담당자를 빼는 것은 edu-fit-frontend 판단이라 거부 코드가 없음
- 배정 취소는 담당자를 해제하고 접수(RECEIVED)로 회귀하며 최초 배정 시각·배정 이력·산출물·댓글을 유지하고 UNASSIGNED 히스토리를 남김
- 담당자가 없는 업무의 배정 취소는 409 TASK_NOT_ASSIGNED로 거부하며 히스토리를 남기지 않음
- 배정·재배정 요청의 comment(content, fileIds)는 원장이 작성한 INTERNAL 댓글로 배정과 같은 트랜잭션에서 저장되어 원장·선생 타임라인에 보임
- 같은 담당자로의 재배정 재요청이어도 comment 댓글은 만들어짐
- comment의 본문·첨부가 모두 비면 400 COMMENT_EMPTY, 타인 파일은 400 FILE_NOT_OWNED, 없는 파일은 404 FILE_NOT_FOUND로 거부하며 배정·히스토리·배정 이력까지 되돌림

### 배정 이력과 열람 범위

- 업무 배정 이력(task_assignment)은 업무에 한 번이라도 배정된 담당자마다 한 행이며 배정·재배정 때 (담당자, 업무) 행이 없으면 추가하고 배정 취소·재배정으로 빠져도 지우지 않음
- 배정 이력 행을 빠뜨리면 현재 담당자도 자기 업무를 404로 받음 — 배정 이력이 선생 업무 열람 범위와 담당 학생 판정의 기준임
- 선생 업무 API는 배정 이력에 본인 행이 있는 철회되지 않은 업무(현재·이전 담당)만 목록·캘린더·상세·히스토리·숨김으로 다룸
- 선생이 담당한 적 없는 업무(한 번도 배정되지 않은 업무 포함)와 철회한 업무는 404 TASK_NOT_FOUND이며 원장도 철회한 업무와 없는 업무를 구분하지 않고 404 TASK_NOT_FOUND로 응답함
- 원장·선생 업무 숨기기·복원(PUT·DELETE /api/{director|teacher}/tasks/:taskId/hidden)은 요청자 본인 목록의 isHidden·hidden 필터에만 반영됨 — 숨김은 회원별 행(task_hidden)임
- 숨기기·복원은 멱등이라 반복해도 성공하며 선생은 이전 담당 업무도 숨길 수 있고 철회한 업무는 404임

### 히스토리 조회

- 업무 히스토리(task_history) 조회는 기록 시각·ID 오름차순 배열임
- 히스토리 문구 속 이름은 기록 시점 값이고 행위자 이름(actor.name)은 현재 회원 이름이며 행위자 회원을 찾지 못하면 actor가 null임

### 신청 내용 대리 수정

- 원장 신청 내용 대리 수정(PATCH /api/director/tasks/:taskId)은 services·subjectAndType·message·axisId·dueDate·noticeFileIds·workFileIds만 받음
- 대리 수정에서 학생 요청 값인 schoolLevel·grade·semester는 400 INVALID_REQUEST로 거부함
- 대리 수정에서 생략한 필드는 바꾸지 않고 axisId·message만 null로 지울 수 있음
- 대리 수정은 현재 값과 실제로 다른 필드만 반영하고 바뀐 필드 이름을 고정 순서로 담은 REQUEST_UPDATED 히스토리 하나를 남김
- 같은 값이나 빈 본문의 대리 수정은 히스토리와 수정 시각을 바꾸지 않음
- noticeFileIds·workFileIds를 보내면 그 분류의 첨부를 배열 순서로 전체 교체하며 업무에 이미 연결된 파일(학생 업로드 포함)은 유지하거나 분류를 옮길 수 있음
- 대리 수정에서 새로 연결하는 파일만 원장 본인 업로드여야 하며 타인 파일은 400 FILE_NOT_OWNED, 없는 파일은 404 FILE_NOT_FOUND임
- 없거나 삭제된 탐구 축(axisId)은 404 INQUIRY_AXIS_NOT_FOUND이고 검증 실패 시 아무것도 바꾸지 않음
- 대리 수정의 입력 상한은 과목 및 유형(subjectAndType) 코드 포인트 1~255자(공백만 불가), 마감 기한(dueDate) 실재하는 YYYY-MM-DD(2026-02-30 거부)임
- 대리 수정의 전달 내용(message) 상한은 UTF-8 65,535 bytes이며 짝 없는 서로게이트도 저장함
- 선생 마감 기한 변경(PATCH /api/teacher/tasks/:taskId/due-date)은 현재 담당자만 가능해 이전 담당자는 403 FORBIDDEN임
- 선생 마감 기한 변경은 값이 바뀔 때만 행위자가 선생인 `신청 내용 수정: 마감 기한` REQUEST_UPDATED 히스토리를 남김

### 상태 변경

- 업무 상태 변경(PUT /api/{teacher|director}/tasks/:taskId/status)은 status로 IN_PROGRESS·COMPLETED만 받고 RECEIVED·소문자·미정의 값·누락·정의되지 않은 필드는 400 INVALID_REQUEST임
- 같은 상태로 바꾸면 상태·히스토리 변경 없이 현재 값으로 200을 응답함
- 상태가 실제로 바뀌면 행위자(상태를 바꾼 선생·원장)를 담은 STATUS_CHANGED 히스토리를 남기고 task.updatedAt도 갱신함
- 상태 변경은 미배정(접수) 업무를 409 TASK_NOT_ASSIGNED로 먼저 거부해 배정 취소된 업무의 이전 담당자도 403이 아닌 409를 받음
- 상태 변경에서 선생은 다른 담당자에게 재배정된 업무에 403 FORBIDDEN, 담당한 적 없거나 철회했거나 없는 업무에 404 TASK_NOT_FOUND를 받음
- 상태 변경에서 원장은 담당자와 무관하게 배정된 업무의 상태를 바꿈
- 상태 변경과 배정 취소가 동시에 오면 업무 행 잠금으로 차례로 처리되어 상태 변경이 먼저일 때만 200이고 뒤이면 409 TASK_NOT_ASSIGNED이며 최종 상태는 접수임

### 산출물

- 산출물 저장(PUT /api/{teacher|director}/tasks/:taskId/deliverable)은 요청 값으로 산출물 전체를 교체하며 keywords·fileIds·followUps는 필수임
- 산출물 저장에서 생략한 title·feedback·career는 null로 저장하고 정의되지 않은 필드는 400 INVALID_REQUEST임
- 산출물의 후속 탐구(task_follow_up)는 요청 배열 순서가 표시 순서(sortOrder 1부터)이고 저장마다 모두 지우고 다시 만들어 id가 새로 매겨짐
- 공유 파일·후속 탐구 파일은 중복을 한 번만 연결하고 요청 순서가 아닌 fileId 오름차순(업로드 순)으로 응답함
- 선생은 현재 담당 업무의 산출물만 저장해 이전 담당·미배정 업무는 403 FORBIDDEN, 담당한 적 없거나 철회했거나 없는 업무는 404 TASK_NOT_FOUND임
- 원장은 담당자·상태와 무관하게 미배정·완료 업무까지 산출물을 대리 저장하며 없거나 철회한 업무만 404임
- 산출물에 새로 연결하는 파일은 저장하는 본인 업로드여야 해 타인 파일·학생 요청 첨부(task_file)는 400 FILE_NOT_OWNED, 없는 파일은 404 FILE_NOT_FOUND임
- 산출물에 이미 연결된 공유 파일·후속 탐구 파일은 업로더와 무관하게 유지하거나 공유 파일과 후속 탐구 사이로 옮길 수 있고 실패한 저장은 기존 산출물을 바꾸지 않음
- 산출물 입력 상한은 title·career·후속 탐구 topic 코드 포인트 255자, feedback·후속 탐구 content UTF-8 65,535 bytes임
- 산출물 keywords는 짝 없는 서로게이트가 없는 문자열 배열, fileIds 원소는 1~4,294,967,295 정수이며 위반은 400 INVALID_REQUEST임
- 산출물 저장은 업무 히스토리를 남기지 않고 업무 수정 시각(task.updatedAt)도 바꾸지 않으며 산출물 updatedAt은 값이 같은 저장에도 저장마다 초 단위 현재 시각으로 갱신됨
- 산출물은 업무당 하나이며 저장 즉시 원장·선생·학생 업무 상세의 deliverable과 원장 업무 수정·학생 요청 등록·수정 응답에 같은 값으로 실림
- 산출물 행이 없으면 deliverable은 null이며 배정이 취소돼도 학생 상세에 남음
- 산출물 파일 항목은 원장·선생 응답에만 uploaderId가 있고 학생 응답에는 없음

## 코드값

원본: `TaskStatus` 전 3종

| 코드 | 뜻 |
|---|---|
| `RECEIVED` | 접수 — 담당자 미배정과 같은 뜻이며 학생이 요청을 등록하면 이 상태로 시작함 |
| `IN_PROGRESS` | 진행 중 |
| `COMPLETED` | 완료 |

원본: `TaskHistoryEventType` 전 5종

| 코드 | 뜻 | 문구 형식 |
|---|---|---|
| `ASSIGNED` | 배정 | `{담당자} 배정` (예: 김선생 배정) |
| `REASSIGNED` | 재배정 | `{이전 담당자} → {새 담당자} 재배정` (예: 김선생 → 이선생 재배정) |
| `UNASSIGNED` | 배정 취소 | `{담당자} 배정 취소` (예: 이선생 배정 취소) |
| `REQUEST_UPDATED` | 신청 내용 수정 | `신청 내용 수정: {바뀐 필드명}` (예: 신청 내용 수정: 요청 서비스, 마감 기한) |
| `STATUS_CHANGED` | 상태 변경 | `상태 변경: {이전 상태} → {새 상태}` (예: 상태 변경: 진행 중 → 완료) |

## 상태와 전이

- 원장이 미배정 업무에 담당자를 배정(PUT /api/director/tasks/:taskId/assignee)하면 접수에서 진행 중(IN_PROGRESS)으로 바뀌고 ASSIGNED 히스토리를 남김
- 최초 배정 시각(firstAssignedAt)은 비어 있을 때만 기록되어 재배정·배정 취소 후 재배정에도 처음 값을 유지함
- 진행 중과 완료 사이 전이는 순서를 강제하지 않고 완료에서 진행 중 회귀를 허용함
- 접수 회귀는 배정 취소로만 도달함

## 절차

- 산출물 저장 후 완료는 edu-fit-frontend가 산출물 저장 API와 상태 변경 API를 순서대로 2회 호출함
