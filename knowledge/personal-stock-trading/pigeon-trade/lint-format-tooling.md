---
description: 린트 규칙을 바꾸거나 oxlint 경고·오류를 처리할 때
---

# 린트 도구

## 규칙

- 런타임 검증 없는 `as T` 단언에는 사유 주석과 oxlint-disable-next-line을 함께 둠 — oxlint가 typescript/no-unsafe-type-assertion을 오류로 판정함
- 새 인터페이스·인자 타입에는 readonly를 붙임 — prefer-readonly-parameter-types 경고를 피함

## 함정

- 응답 DTO에서 인자 없는 Decimal#toFixed() 호출에 뜨는 unicorn(require-number-to-fixed-digits-argument) 경고는 그대로 둠(의도된 동작) — 인자를 주면 소수 자릿수가 고정되어 '71500.5'가 '71500.50000000'이 됨

## 결정

- 원조 eslint-config-airbnb는 기각함 — flat config 미지원·유지보수 중단 상태임
- eslint-config-airbnb-extended와 typescript-eslint strictTypeChecked는 기각함 — ESLint 재도입 비용이 큼
