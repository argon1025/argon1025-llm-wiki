---
description: HTTP 부품(DTO·필터·인터셉터)을 vitest로 단위 테스트할 때
type: convention
---

# HTTP 부품 테스트

## 원인

- Nest 앱을 띄운 테스트에서는 생성자 주입과 ValidationPipe의 DTO 판별이 동작하지 않음 — vitest가 esbuild로 변환해 데코레이터 메타데이터(emitDecoratorMetadata)를 내보내지 않음

## 테스트 방식

- HTTP 부품은 Nest 앱 e2e 없이 부품별 단위 테스트로 검증함(DTO는 validate(), 필터·인터셉터는 직접 생성) — 전역 등록·파이프 연결은 자동 테스트가 없음
- SWC(unplugin-swc) 도입과 Nest 앱 e2e는 조립 오류가 반복될 때 검토함

## 구현 세부

- 직렬화 인터셉터(ClassSerializerInterceptor)를 spec에서 직접 쓸 때는 { transformerPackage: classTransformer }를 넘겨야 함 — 생성자가 class-transformer를 비동기 로딩함
