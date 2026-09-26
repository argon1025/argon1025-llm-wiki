---
description: 트레이드 손익의 정본이 무엇인지 확인할 때
type: domain
---

# 지갑 트레이드 손익

## 적용 대상

- pigeon-trade 레포에서만 참

## 용어

- 청산 트레이드(CLOSED) 손익의 정본은 구간 현금 증감(cash_delta) 합 — 매도 주문 실현 손익(realized_pnl) 합과 같으나 평균 단가를 decimal(24,8)로 반올림 저장해 소수 8자리 이하에서 어긋날 수 있음
