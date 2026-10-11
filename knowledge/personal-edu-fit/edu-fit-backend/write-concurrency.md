---
description: 동시 요청의 행 잠금·유니크 위반·순서 값 충돌을 다룰 때
---

# 쓰기 동시성

## 규칙

- consulting에서 업무의 하위 연결 행(task_file 등)을 조회한 뒤 지우고 다시 넣는 쓰기(StudentTaskService.updateTask)는 em.transactional 안에서 업무 행을 LockMode.PESSIMISTIC_WRITE로 먼저 읽음 — 동시 요청의 첨부 중복 연결을 막음
- 철회(StudentTaskService.deleteTask)·숨김(hideTask)처럼 단일 행만 갱신하거나 멱등인 쓰기는 업무 행을 잠그지 않음
- 원장 학생 등록·수정(DirectorStudentService.flush)의 LOGIN_ID_DUPLICATED는 사전 count 없이 flush 중 UniqueConstraintViolationException을 409로 바꾸는 한 경로로 판정함 — 동시 등록까지 처리함
- 원장 선생 서비스(DirectorTeacherService.flush)는 유니크 위반의 sqlMessage에 user_login_id_unique가 있을 때만 409 LOGIN_ID_DUPLICATED로 바꾸고 나머지는 그대로 던짐 — teacher_subject 복합 PK 충돌을 로그인 ID 중복으로 볼 수 없음

## 함정

- 업무 숨기기(StudentTaskService.hideTask)의 em.upsert(TaskHidden, ...)를 em.create 후 flush로 바꾸면 이미 숨긴 업무에서 UniqueConstraintViolationException이 남 — upsert는 동시 요청에도 PK 중복 오류가 나지 않음
- 학생 서비스(DirectorStudentService.flush)는 user·student_profile에 user.login_id 외 유니크 키가 생기면 제약 이름으로 유니크 위반을 구분해야 함 — 유니크 키가 user.login_id 하나라는 전제로 모든 유니크 위반을 LOGIN_ID_DUPLICATED로 바꿈
- 같은 선생의 담당 과목을 동시에 교체하면(DirectorTeacherService.replaceSubjects) teacher_subject PK 충돌·교착으로 500이 될 수 있음 — 행 잠금 없이 기존 행을 읽고 차이만 삭제·추가하는 의도된 단순화
- 탐구 축을 동시에 추가하면(DirectorInquiryAxisService.createInquiryAxis) 같은 sort_order가 생길 수 있음 — 행 잠금 없이 최대 sort_order를 읽고 +1로 저장하는 의도된 단순화
- 탐구 축의 sort_order가 같아도 목록은 id 2차 정렬로 순서가 고정되고 다음 순서 변경(reorderInquiryAxes)이 1부터 다시 매겨 해소함

## 결정

- 담당 과목 동시 교체가 운영 문제가 되면 수정 시 선생 user 행을 PESSIMISTIC_WRITE로 잠근 뒤 담당 과목을 읽음
- 탐구 축 동시 추가가 운영 문제가 되면 추가 시 행 잠금을 도입함
