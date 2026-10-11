---
description: MikroORM으로 엔티티를 저장·삭제하거나 하위 행을 교체할 때
---

# ORM 쓰기 함정

## 적용 대상

- edu-fit-backend의 MikroORM 엔티티 저장·삭제 코드

## 규칙

- 연결 행을 읽지 않고 em.nativeUpdate로 일괄 갱신할 때는 바꿀 값에 updatedAt을 직접 넣음(수업 설정 삭제의 연결 수업 소프트 삭제, DirectorCourseService.deleteCourse) — nativeUpdate는 onUpdate를 거치지 않음
- 하위 행을 일괄 삭제하고 같은 엔티티의 새 행을 만드는 저장(산출물 공유 파일·후속 탐구, 성취도 항목·행 값, 업무 요청 첨부·댓글 첨부 연결)은 em.remove 뒤 em.flush()를 먼저 부르고 새 행을 만들어 두 번째 flush로 저장함 — 한 flush에 담으면 UnitOfWork.computeChangeSets가 삭제 수 × 생성 수만큼 대조해 1MB 본문 규모에서 이벤트 루프가 수 초에서 10분 넘게 멈춤
- 업무 요청 첨부(updateTask)·업무 댓글 첨부(updateComment) 교체는 지울 연결(task_file·task_comment_file)이 있을 때만 선 flush를 함(첨부를 바꾸지 않는 수정은 flush 1회)
- 업무 요청 첨부 교체(updateTask)는 새 연결(task_file)을 첫 flush 뒤 요청 순서로 만듦 — task_file id 오름차순이 첨부 순서임
- 선생 담당 과목 교체(DirectorTeacherService.replaceSubjects)는 삭제 선 flush 없이 한 flush를 유지함 — 원장이 만든 활성 과목 ID만 통과해 교체 행 수가 과목 수로 한정됨
- 탐구 이력 수정의 후속 탐구 교체(DirectorInquiryRecordService.updateRecord)는 em.nativeDelete(InquiryRecordFollowUp, { record }) 한 문장으로 기존 행을 지우고(파일 연결은 FK cascade) 새 행만 em.create로 만들어 한 번 flush함 — em.remove와 em.create를 한 flush에 담으면 삭제 수 × 생성 수 대조로 이벤트 루프가 멈춤
- PATCH 요청 반영(탐구 이력 수정)은 em.assign(entity, data, { ignoreUndefined: true })로 전달한 필드만 바꿈 — em.assign은 기본값이면 undefined도 대입해 null 허용 컬럼을 지움
- 리포트 작성·수정(DirectorReportService·TeacherReportService의 createReport·updateReport) 응답을 저장값과 일치시켜야 하면 저장 후 em.refresh로 행을 다시 읽어 응답함 — 요청 문자열을 그대로 실으면 짝 없는 서로게이트가 응답에는 원래 값, 이후 조회에는 mysql2가 바꿔 저장한 U+FFFD로 나옴

## 함정

- 업무 요청 첨부·댓글 첨부 교체는 삭제 선 flush 뒤에도 1MB 본문의 최대 개수(업무 첨부 16만 5,666개, 댓글 첨부 14만 9,794개) 교체에 7.0~8.7초, 이벤트 루프 최대 지연 1.4~1.5초가 남음(의도적 단순화)
- 복합 PK 하위 행을 교체할 때 기존 행을 모두 em.remove한 뒤 em.create하지 않고 빠진 행만 지우고 새 행만 넣음(댓글 첨부) — MikroORM flush가 insert를 delete보다 먼저 실행해 (comment_id, file_id) PK 중복 오류가 남
- DB FK on delete cascade로 하위 행이 지워지는 삭제(성취도 표·항목·행) 뒤 같은 요청에서 하위 엔티티를 다시 쓸 때는 DB에서 조회함 — 엔티티에 역방향 컬렉션이 없어 MikroORM이 cascade로 지워진 하위 행을 identity map에 남김
- LuxonDateTimeType 속성(updatedAt 등)은 em.assign으로 바꾸지 않고 entity.updatedAt = DateTime.utc().startOf('second')처럼 직접 대입함 — em.assign은 그 값을 string 타입으로 검사해 ValidationError(500)를 던짐
