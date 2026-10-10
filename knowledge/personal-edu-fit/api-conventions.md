---
description: 역할별 API를 나누거나 응답·페이지네이션·시각 규약을 정할 때
---

# API 규약

## 적용 대상

- API를 제공하는 edu-fit-backend, 응답을 받아 화면에 표시하는 edu-fit-frontend

## 규칙

- 계정당 역할은 원장·선생·학생 중 하나임
- API는 원장·선생·학생 역할별로 분리해 제공하고, 인증·파일·탐구 축 조회처럼 역할 차이가 없는 기능만 공통 API로 둠
- 기능이 같아도 역할이 다르면 API를 역할별로 따로 둠 — 역할마다 바뀌는 원인이 다름
- 역할 범위 밖 리소스 접근은 존재하지 않는 리소스와 같은 응답으로 거부해 존재 여부를 노출하지 않음
- 조회는 되나 쓰기 권한이 없을 때(타인 작성 기록 수정 등)만 권한 없음 응답으로 거부함
- 목록 조회는 페이지 번호·페이지 크기로 나누는 오프셋 페이지네이션이며 전체 건수를 함께 돌려줌
- DB와 edu-fit-backend는 모든 시각을 UTC로 다루고, 현지 시각 표시 변환은 edu-fit-frontend가 함
- edu-fit-backend 응답은 공통 envelope(success·responseCode·message·data)이며 실패 시 data는 null임
