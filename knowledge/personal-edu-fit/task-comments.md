---
description: 업무 댓글을 쓰거나 고치거나 내부 댓글 노출 범위를 다룰 때
---

# 업무 댓글

## 적용 대상

- 댓글을 작성·표시하는 edu-fit-frontend, 댓글을 저장하고 학생 응답에서 내부 댓글을 걸러 내는 edu-fit-backend

## 규칙

- 업무 댓글의 공개 범위는 공개 댓글(업무 관련자 전체)과 내부 댓글(원장·담당자 전용, 학생 비노출) 두 가지임
- 업무 댓글 유형은 본문(+선택 첨부) 댓글과 본문 없이 첨부만 있는 파일 엔트리 두 가지뿐이며 시스템 기록 유형은 없음
- edu-fit-backend가 업무 댓글 유형을 본문 유무로 정함
- 학생 대상 API 응답은 내부 댓글과 그 첨부를 빈 값으로도 두지 않고 필드 자체를 싣지 않음
- 업무 댓글 수정·삭제는 작성자 본인만 가능함
- 원장도 타인이 단 업무 댓글의 첨부를 단건 삭제할 수 없음

### 작성과 본문

- 댓글 본문(content)이 생략·null·빈 문자열·공백뿐이면 null로 저장하고 공백 아닌 문자가 있으면 받은 값을 그대로 저장하며, 유형(type)은 요청이 아니라 이 저장 본문의 유무로 정함
- 결과 본문과 첨부(fileIds)가 모두 비면 400 COMMENT_EMPTY로 거부함
- 댓글 작성·수정과 배정·재배정 comment의 본문(content)은 UTF-8 65,535 bytes(한글 21,845자) 이하이고 넘으면 400 INVALID_REQUEST이며 짝 없는 서로게이트도 받음
- 댓글 작성은 POST /api/{student|teacher|director}/tasks/:taskId/comments이며 201로 응답함
- 댓글 수정·삭제는 PATCH·DELETE /api/{student|teacher|director}/task-comments/:commentId이며 삭제 응답 data는 null임
- 원장·선생 작성은 공개 범위(visibility)가 필수이고 학생 작성은 PUBLIC 고정이라 학생이 visibility를 보내면 400 INVALID_REQUEST임
- 댓글 첨부(fileIds)에 없는 파일(404 FILE_NOT_FOUND)이나 타인이 업로드한 파일(400 FILE_NOT_OWNED)이 하나라도 섞이면 나머지 파일도 연결하지 않고 요청 전체를 거부함
- 작성 때 fileIds는 필수이고 원소는 1~4294967295 정수이며 중복은 한 번만 연결함
- 댓글 요청은 업무 범위(404)·작성자(403)를 본문·첨부 판정(400 COMMENT_EMPTY, 404 FILE_NOT_FOUND, 400 FILE_NOT_OWNED)보다 먼저 확인함 — 권한 없는 요청이 본문과 무관하게 같은 응답을 받음

### 조회

- 댓글 타임라인(GET /api/{student|teacher|director}/tasks/:taskId/comments)은 목록 오프셋 페이지네이션 규약과 달리 페이지·전체 건수 없이 작성 시각·ID 오름차순 배열을 반환함(의도된 동작)
- 각 댓글의 첨부는 요청 순서가 아니라 파일 ID 오름차순임
- 원장·선생 타임라인은 visibility 쿼리(PUBLIC|INTERNAL)로 거르고 생략하면 전체이며 그 밖의 값은 400임
- 학생 댓글 응답에는 visibility 필드가 없고 학생 타임라인은 visibility 쿼리를 보내도 무시함
- 댓글 응답의 작성자(author)는 id·name·role이며 name은 작성 당시가 아니라 현재 회원 이름임
- 댓글 createdAt·updatedAt은 초 단위 UTC ISO 8601이며 작성·수정 응답은 이후 타임라인 항목과 같음
- 첨부의 uploaderId는 선생·원장 응답에만 있음

### 역할별 권한

- 학생은 본인 요청 업무의 PUBLIC 댓글만 조회·작성하고 다른 학생·철회·없는 업무는 404 TASK_NOT_FOUND임
- 학생이 INTERNAL 댓글이나 다른 학생 요청의 댓글 ID로 수정·삭제하면 없는 댓글과 같은 404 COMMENT_NOT_FOUND임
- 선생은 현재 담당자(task.assigneeId)가 본인인 업무에서만 댓글을 조회·작성·수정·삭제함
- 재배정·배정 취소로 현재 담당자가 아니게 된 선생은 그 업무의 댓글 조회·작성이 404 TASK_NOT_FOUND, 본인 댓글의 수정·삭제가 404 COMMENT_NOT_FOUND이며 댓글은 그대로 남음(의도된 동작)
- 원장은 철회되지 않은 모든 업무의 댓글을 조회하고 PUBLIC·INTERNAL로 작성하며 없거나 철회한 업무는 404 TASK_NOT_FOUND임
- 타인이 쓴 댓글의 수정·삭제는 원장을 포함한 모든 역할에서 403 FORBIDDEN임
- 경로가 댓글 ID인 수정·삭제는 없는 댓글·철회한 업무의 댓글·역할 범위 밖 댓글이면 404 COMMENT_NOT_FOUND임 — 존재 여부를 노출하지 않음

### 수정과 삭제

- 댓글 수정(PATCH)에서 content를 생략하면 본문 유지, null·빈 문자열·공백이면 본문 삭제임
- 댓글 수정으로 본문을 지워 첨부만 남기면 COMMENT에서 FILE로, 본문을 넣으면 FILE에서 COMMENT로 바뀌며 FILE 엔트리도 작성자가 수정할 수 있음 — 최종 요구사항 문서 QST-19와 다른 의도된 동작임
- 댓글 수정에서 fileIds를 생략하면 첨부 유지, 전달하면 첨부 전체 교체이며 fileIds null은 400임
- 댓글 수정 요청에 visibility를 보내면 400 INVALID_REQUEST임 — 공개 범위는 작성 후 바꿀 수 없음
- 댓글 수정은 결과 본문·첨부가 현재와 다를 때만 수정 시각(updatedAt)을 갱신하고, 빈 요청 본문 {}나 같은 값은 200으로 현재 댓글을 그대로 응답함
- 댓글 삭제는 하드 삭제이며 첨부 연결 행도 함께 지움

## 코드값

원본: `TaskCommentVisibility` 전 2종

| 코드 | 뜻 |
|---|---|
| `PUBLIC` | 업무 관련자 전체(원장·담당 선생·요청 학생)가 보는 댓글 |
| `INTERNAL` | 원장·담당자 전용 댓글이며 학생에게 어떤 응답으로도 노출하지 않음 |

원본: `TaskCommentType` 전 2종

| 코드 | 뜻 |
|---|---|
| `COMMENT` | 본문이 있고 첨부는 선택인 댓글 |
| `FILE` | 본문 없이 첨부만 있는 댓글이며 본문(content)은 null |
