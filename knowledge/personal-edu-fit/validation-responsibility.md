---
description: API 요청 검증을 백엔드와 프론트엔드 중 어디서 할지 정할 때
---

# 검증 책임 분담

## 적용 대상

- 요청을 검증하는 edu-fit-backend, 저장 전에 입력을 판단하는 edu-fit-frontend

## 규칙

- edu-fit-backend는 아래 표에서 edu-fit-backend로 적은 보안·데이터 성립·불가능한 전이만 검증함 — 이후 요구사항 변경에 백엔드가 유연하게 대응하기 위함
- 아래 표에서 edu-fit-frontend로 적은 항목은 edu-fit-backend가 막지 않으므로 edu-fit-frontend가 저장 전에 판단함

| 검증 항목 | 판단하는 레포 |
|---|---|
| 보안(인증·역할·소유권, 학생 응답의 내부 정보 차단) | edu-fit-backend |
| 데이터 성립(형식, 엔티티 성립 필수 필드, 존재하지 않는 ID, 시작 > 종료 같은 순서 모순, 자기 참조, 로그인 ID 중복) | edu-fit-backend |
| 불가능한 전이 | edu-fit-backend |
| 이름·조합 중복 | edu-fit-frontend |
| 점수·값 범위 | edu-fit-frontend |
| 개수 제한 | edu-fit-frontend |
| 조건부 필수(요청 서비스 1개 이상, 완료 시 제목 필수) | edu-fit-frontend |
| 상태별 편집 제한(학생 접수 상태만 수정, 완료 건 대리 수정 차단) | edu-fit-frontend |
