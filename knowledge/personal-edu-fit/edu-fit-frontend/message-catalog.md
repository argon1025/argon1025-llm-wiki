---
description: 화면에 보이는 액션 결과·안내 문구를 추가하거나 고칠 때
---

# 메시지 카탈로그

## 적용 대상

- edu-fit-frontend의 사용자 노출 액션 결과·안내 문구

## 규칙

- 사용자에게 보이는 액션 결과·안내 문구는 도메인별 키로 모은 메시지 카탈로그(MESSAGES, COM-04)에 두고, 화면 코드는 문구를 직접 쓰지 않고 키로 참조함
- 화면 이슈가 문구를 추가할 때는 MESSAGES에 키를 추가함
- 업무 세그먼트(TaskSegment)로 색인하는 안내 문구(MESSAGES.studentTasks.emptyHint)는 사용처에서 Partial<Record<TaskSegment, string>>로 받아 색인함 — MESSAGES는 shared 코드라 features의 TaskSegment를 import하지 않음
