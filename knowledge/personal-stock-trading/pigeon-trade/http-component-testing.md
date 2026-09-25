---
description: HTTP 부품을 테스트하거나 vitest 데코레이터 메타데이터 문제를 다룰 때
type: convention
---

# HTTP 부품 테스트 방식

## 제약

- vitest는 esbuild 변환이라 데코레이터 메타데이터(emitDecoratorMetadata)를 내보내지 않음 — Nest 앱을 띄운 테스트에서 생성자 주입·ValidationPipe의 DTO 판별·@ApiProperty 타입 추론이 동작하지 않음

## 규칙

- HTTP 부품은 부품별 단위 테스트로 검증함(DTO는 plainToInstance + validate, 필터·인터셉터는 직접 생성, 파이프는 metatype을 직접 넘김) — SWC(unplugin-swc) 도입과 Nest 앱 e2e는 두지 않고 조립 오류가 반복되면 재검토
- 전역 등록·파이프 연결과 Swagger 문서는 자동 테스트 없이 앱을 기동해 수동으로 확인함
