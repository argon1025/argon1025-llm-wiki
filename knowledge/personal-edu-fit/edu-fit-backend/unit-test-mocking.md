---
description: 서비스 단위 테스트에서 vitest spy를 mockReset으로 초기화할 때
---

# 단위 테스트 목 초기화

## 적용 대상

- edu-fit-backend의 서비스 단위 테스트 (예: StudentTaskService)

## 함정

- em.upsert·em.nativeDelete 같은 DB 쓰기 메서드 spy는 mockReset() 뒤 mockResolvedValue를 다시 걸어야 함 — vitest 4의 mockReset()은 spy를 원래 구현으로 되돌림
- mockReset()한 DB 쓰기 메서드 spy에 mockResolvedValue를 다시 걸지 않으면 더미 MikroORM 설정의 localhost:3306 접속을 시도해 ECONNREFUSED로 실패함
