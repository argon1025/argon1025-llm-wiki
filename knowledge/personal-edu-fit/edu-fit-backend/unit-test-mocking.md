---
description: 서비스 단위 테스트에서 vitest spy로 조회 결과를 대체하거나 mockReset으로 초기화할 때
---

# 단위 테스트 목 초기화

## 적용 대상

- edu-fit-backend의 서비스 단위 테스트 (예: StudentTaskService)

## 규칙

- 역할 업무 서비스(DirectorTaskService·StudentTaskService·TeacherTaskService) 단위 테스트는 산출물 서비스의 findDeliverable을 mockResolvedValue(null)로 기본 대체함 — 산출물 조회도 같은 em.findOne을 써서 업무 행으로 고정한 목이 산출물로 읽힘
- 서비스 단위 테스트에서 상태가 다른 엔티티(업무 등)를 차례로 돌려줄 때는 엔티티마다 PK를 다르게 둠 — 같은 forked EntityManager로 같은 PK의 엔티티를 em.create로 여러 번 만들면 identity map에 먼저 등록된 엔티티가 돌아와 뒤 값이 반영되지 않음

## 함정

- em.upsert·em.nativeDelete 같은 DB 쓰기 메서드 spy는 mockReset() 뒤 mockResolvedValue를 다시 걸어야 함 — vitest 4의 mockReset()은 spy를 원래 구현으로 되돌림
- mockReset()한 DB 쓰기 메서드 spy에 mockResolvedValue를 다시 걸지 않으면 더미 MikroORM 설정의 localhost:3306 접속을 시도해 ECONNREFUSED로 실패함
