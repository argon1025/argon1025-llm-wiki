---
description: 동시 요청의 행 잠금·유니크 위반·순서 값 충돌을 다룰 때
---

# 쓰기 동시성

## 규칙

### 공통

- 비관적 잠금 조회는 em.transactional의 첫 조회로 둠 — 트랜잭션 안에서 잠금 없이 먼저 읽으면 MySQL REPEATABLE READ 스냅샷이 고정돼 잠금 대기 중 커밋된 변경을 못 봄
- 잠글 행의 상위 ID(댓글의 업무 ID, 성취도 행의 표 ID)가 필요하면 트랜잭션 밖에서 먼저 읽음 — 잠금 조회를 트랜잭션의 첫 조회로 두기 위함

### consulting

- consulting에서 업무의 하위 연결 행(task_file 등)을 조회한 뒤 지우고 다시 넣는 쓰기(StudentTaskService.updateTask)는 em.transactional 안에서 업무 행을 LockMode.PESSIMISTIC_WRITE로 먼저 읽음 — 동시 요청의 첨부 중복 연결을 막음
- 철회(StudentTaskService.deleteTask)·숨김(hideTask)처럼 단일 행만 갱신하거나 멱등인 쓰기는 업무 행을 잠그지 않음
- 업무 배정(DirectorTaskService.assignTask)·배정 취소·신청 내용 대리 수정·마감일 변경·산출물 저장·상태 변경은 em.transactional 안에서 업무 행을 PESSIMISTIC_WRITE로 먼저 읽음 — 잠금이 없으면 동시 배정의 ASSIGNED 히스토리 중복·task_assignment PK 중복 500, 동시 첫 산출물 저장의 task_deliverable PK 중복 500, 동시 상태 변경 히스토리 중복이 생김
- 선생의 업무 쓰기(마감일 변경·산출물 저장·상태 변경, findTaskInScope)는 업무 행을 잠근 뒤 배정 이력(없으면 404 TASK_NOT_FOUND)·현재 담당자(아니면 403 FORBIDDEN) 순으로 확인함
- 댓글 수정·삭제(DirectorTaskCommentService.lockOwnComment)는 트랜잭션 밖에서 findOne(TaskComment, commentId)로 업무 ID를 얻고, 트랜잭션 첫 조회로 업무 행을 PESSIMISTIC_WRITE로 잠근 뒤 댓글을 refresh: true로 다시 읽음 — refresh 없이 다시 읽으면 같은 댓글의 동시 첨부 교체·수정과 삭제가 500이 됨
- 원장·학생 댓글 작성은 업무 행을 잠그지 않음 — 새 댓글 행만 넣음
- 선생 댓글 작성(TeacherTaskCommentService)은 업무 행을 잠금 — 현재 담당자 판정을 재배정과 직렬화함

### course

- course 도메인에서 설정과 수업을 함께 쓰는 쓰기는 설정 행 → 수업 행 순으로 잠금
- 수업 생성은 설정 행을 PESSIMISTIC_READ(FOR SHARE)로, 설정 삭제(DirectorCourseService.deleteCourse)는 설정 행을 PESSIMISTIC_WRITE로 읽은 뒤 연결 수업을 갱신함 — 잠금을 빼면 삭제된 설정에 활성 수업이 남거나 삭제가 교착 500이 됨
- 수업 수정(DirectorLessonService.updateLesson)은 수업 행만 { id, deletedAt: null } 조건의 PESSIMISTIC_WRITE로 잠그고 소속 설정은 잠금 없이 따로 확인함 — 설정 조건을 조인해 잠그면 설정 행까지 잠겨 설정 삭제와 반대 순서로 교착됨
- 수업 수정에서 잠금을 빼면 startAt만·endAt만 바꾸는 동시 수정이 둘 다 성공해 endAt ≤ startAt이 저장됨
- 수업 기록 수정·삭제(DirectorLessonNoteService.lockNote)는 기록 행만 PESSIMISTIC_WRITE로 잠그고 귀속 수업·설정은 잠금 없이 em.count로 확인함 — 수업·설정을 조인한 잠금은 설정 삭제(설정 → 수업 순 잠금)와 교착될 수 있음
- 수업 기록 수정·삭제에서 잠금 없이 조회 후 삭제하면 동시 삭제가 모두 200이 됨
- 수업 기록 작성(DirectorLessonNoteService.createNote)은 수업 행을 잠그지 않고 em.count로 수업 존재만 확인한 뒤 insert함 — 기록은 수업·설정 소프트 삭제 후에도 보존되는 하위 행이라 지킬 불변식이 없고 FK 확인이 수업 행 공유 잠금만 잡아 설정 → 수업 순 잠금과 순환하지 않음

### achievement

- 성취도 항목·행·값 쓰기와 표 삭제(DirectorAchievementService.lockTable)는 트랜잭션 첫 조회로 표 행을 PESSIMISTIC_WRITE(refresh: true)로 잠근 뒤 하위 행을 읽고 바꿈
- 성취도 표 행 잠금을 빼면 같은 학기 행 동시 추가 500, FREE 행 sortOrder 중복, 행 기준 변경과 행 추가 동시 성공, 값 동시 교체 500, 정의 동시 교체의 항목 중복, 같은 표 동시 삭제 이중 200이 재현됨
- 성취도 행 수정·삭제(findRowTableId)는 트랜잭션 밖에서 행을 읽어 표 ID를 얻은 뒤 표 행을 잠그고, 잠금 뒤 행·값은 refresh: true로 다시 읽음
- 선생 성취도 표·행 경로(TeacherAchievementService.lockTable)는 범위 밖 요청도 범위 판정(TeacherScopeExternalService.isInScope) 동안 표 행 쓰기 잠금을 잡음 — 표 행을 잠근 뒤 잠근 표의 studentId로 범위를 판정함

### report

- 리포트 수정·삭제(DirectorReportService.lockReport)는 em.transactional 첫 조회로 리포트 행을 LockMode.PESSIMISTIC_WRITE로 읽어 같은 리포트의 뒤 요청이 404 REPORT_NOT_FOUND를 받게 함 — 잠금 없이 조회하면 동시 삭제 두 건이 모두 200이 됨

### member

- 선생 담당 학생 학년 수정(TeacherStudentService.updateStudentGrade)은 행 잠금을 두지 않음 — student_profile 한 행의 값만 교체함
- 원장 학생 등록·수정(DirectorStudentService.flush)의 LOGIN_ID_DUPLICATED는 사전 count 없이 flush 중 UniqueConstraintViolationException을 409로 바꾸는 한 경로로 판정함 — 동시 등록까지 처리함
- 원장 선생 서비스(DirectorTeacherService.flush)는 유니크 위반의 sqlMessage에 user_login_id_unique가 있을 때만 409 LOGIN_ID_DUPLICATED로 바꾸고 나머지는 그대로 던짐 — teacher_subject 복합 PK 충돌을 로그인 ID 중복으로 볼 수 없음

## 함정

- 트랜잭션 밖에서 읽은 엔티티를 em.transactional 안에서 refresh·비관적 lockMode 없이 findOne하면 DB를 조회하지 않음 — em.transactional의 clear: false fork가 identity map을 복사함
- 잠금 없이 조회한 행을 em.remove 후 flush로 지우면 다른 요청이 먼저 지운 행이어도 성공함 — MikroORM flush는 DELETE 영향 행 수를 확인하지 않음
- 업무 숨기기(StudentTaskService.hideTask)의 em.upsert(TaskHidden, ...)를 em.create 후 flush로 바꾸면 이미 숨긴 업무에서 UniqueConstraintViolationException이 남 — upsert는 동시 요청에도 PK 중복 오류가 나지 않음
- 학생 서비스(DirectorStudentService.flush)는 user·student_profile에 user.login_id 외 유니크 키가 생기면 제약 이름으로 유니크 위반을 구분해야 함 — 유니크 키가 user.login_id 하나라는 전제로 모든 유니크 위반을 LOGIN_ID_DUPLICATED로 바꿈
- 같은 선생의 담당 과목을 동시에 교체하면(DirectorTeacherService.replaceSubjects) teacher_subject PK 충돌·교착으로 500이 될 수 있음 — 행 잠금 없이 기존 행을 읽고 차이만 삭제·추가하는 의도된 단순화
- 탐구 축을 동시에 추가하면(DirectorInquiryAxisService.createInquiryAxis) 같은 sort_order가 생길 수 있음 — 행 잠금 없이 최대 sort_order를 읽고 +1로 저장하는 의도된 단순화
- 탐구 축의 sort_order가 같아도 목록은 id 2차 정렬로 순서가 고정되고 다음 순서 변경(reorderInquiryAxes)이 1부터 다시 매겨 해소함

## 결정

- 담당 과목 동시 교체가 운영 문제가 되면 수정 시 선생 user 행을 PESSIMISTIC_WRITE로 잠근 뒤 담당 과목을 읽음
- 탐구 축 동시 추가가 운영 문제가 되면 추가 시 행 잠금을 도입함
