---
description: 학생 요청 상세 화면의 404·철회 후 목록 이동이나 진행 요약 담당자 표기를 고칠 때
---

# 학생 요청 상세 화면

## 적용 대상

- edu-fit-frontend의 학생 요청 상세 화면(StudentTaskDetailScreen)

## 규칙

- 상세·댓글 조회가 404 TASK_NOT_FOUND면 effect는 토스트와 목록(/student/tasks) 이동만 하고 leaving 상태·캐시는 건드리지 않음 — effect 안에서는 setState가 금지됨
- 철회 성공이나 mutation의 404 TASK_NOT_FOUND면 콜백이 leaving 상태로 상세·댓글 재조회를 막고 상세 캐시를 지운(removeQueries) 뒤 목록(/student/tasks)으로 이동함
- 댓글 쿼리 키(studentTaskDetailKeys.comments)는 상세 키(studentTaskDetailKeys.detail) 아래에 둠 — 상세 키 캐시 제거 한 번에 댓글 캐시도 지워짐
- 목록으로 이동할 때 목록 캐시(studentTaskKeys.all)는 refetchType none으로 무효화만 표시해 이동 뒤 마운트할 때 불러오게 함 — 지금 재조회하면 이 요청이 다시 404를 받음
- 요청 상세 진행 요약(ProgressSummary)은 담당자 라벨을 '담당 선생님'이 아닌 '담당자'로 표기함 — 원장도 요청의 담당자가 될 수 있음
