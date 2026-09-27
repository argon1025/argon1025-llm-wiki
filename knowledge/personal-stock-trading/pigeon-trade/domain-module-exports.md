---
description: 도메인 모듈 외부 노출 구조와 검증 방법을 정할 때
type: convention
---

# 도메인 모듈 외부 노출 컨벤션

## 규칙

- 도메인 모듈 외부 노출은 위임 전용 외부 노출 서비스({Domain}ExternalService) 하나로 두고 토큰+인터페이스·추상 클래스 토큰·DI 파사드 설계안은 쓰지 않음 — 과한 복잡도로 기각

## 검증

- 모듈 연결 변경은 앱 로컬 기동 후 실제 호출(예: /mcp 도구 호출)로 확인함 — unit·e2e spec은 서비스를 new로 직접 생성해 모듈 providers·exports 누락을 잡지 못함
