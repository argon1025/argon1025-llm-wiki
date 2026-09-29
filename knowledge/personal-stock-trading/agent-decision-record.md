---
description: 에이전트 판단을 기록·조회하거나 판단 유형·action을 다룰 때
---

# 에이전트 판단 기록

## 적용 대상

- 판단 기록 API(POST /api/wallets/:walletId/decisions)와 판단 저장·조회를 가진 pigeon-trade

## 규칙

### 기록

- 판단 기록 API는 주문을 실행하지 않고 에이전트가 먼저 체결한 주문의 orderId를 참조로만 받음
- 판단 기록 API는 유형(type)·action·orderId 조합을 검증하지 않아 조합이 어긋나도 저장함 — 기록 용도라 조합을 강제하지 않음
- 판단 기록 API는 orderId가 있으면 같은 지갑의 주문인지(아니면 404 ORDER_NOT_FOUND)와 주문 종목이 요청 종목(symbol)과 같은지(아니면 400 INVALID_REQUEST)만 확인함
- 판단 기록은 지갑(pigeon_wallet) 단위로 남아 지갑이 에이전트 식별을 겸함(경로 /api/wallets/:walletId/decisions) — 에이전트 엔티티가 없음
- 판단 기록은 기록한 에이전트(Jev·메인 에이전트)를 주체 필드 없이 판단 유형(type)으로 구분함
- 진입 매수 뒤의 보유 유지 판단(HOLD)은 그 진입 매수 주문의 orderId를 참조함 — 주문별 판단 목록으로 진입 이후 판단을 조회함
- 보유 없는 관망 판단(WAIT)은 orderId 없이 기록함 — 참조할 진입 주문이 없음
- 판단 기록은 전략을 이름·버전(strategy·strategyVersion)으로만 남기고 프롬프트 원문은 저장하지 않음 — 에이전트가 원문을 다시 출력하면 토큰 비용이 들고 원문과 어긋날 수 있음
- 판단 기록의 주입 컨텍스트(context)는 형식 없는 자유 텍스트(LONGTEXT)이며 서버가 지표를 자동 조회해 채우지 않음 — 컨텍스트 지표 구성 부분이 아직 없음

### 조회

- 한 주문에 판단이 여러 건 달릴 수 있어(agent_decision.order_id 비유니크) 주문별 판단은 목록의 orderId 필터로 조회함
- 판단 목록은 context를 제외하고 상세 조회(GET /api/wallets/:walletId/decisions/:decisionId)에서만 context를 반환함
- 판단 목록은 id 내림차순(최신순)이며 더 과거는 응답 마지막 id를 beforeId(exclusive)로 넘겨 조회함
- 종목 직전 판단은 symbol·type과 limit=1로 조회함
- 판단 목록의 limit은 1~100(기본 20)임
- 판단 목록의 type·action 필터는 쉼표 구분 또는 반복 파라미터로 받음

## 코드값

### 판단 유형

원본: `AgentDecisionType` 전 2종

| 코드 | 뜻 |
|---|---|
| `REGIME` | 주기적으로 호출되는 Jev의 국면 판단(메인 에이전트를 깨울지 여부) |
| `TRADE` | 거래 담당 메인 에이전트의 거래 처리(BUY·SELL·HOLD·WAIT) |

### 판단 action

원본: `AgentDecisionAction` 전 6종

| 코드 | 뜻 |
|---|---|
| `WAKE` | REGIME 판단에서 Jev가 메인 에이전트를 깨움 |
| `SKIP` | REGIME 판단에서 Jev가 메인 에이전트를 깨우지 않음 |
| `BUY` | TRADE 판단에서 매수 |
| `SELL` | TRADE 판단에서 매도 |
| `HOLD` | TRADE 판단에서 보유 유지 |
| `WAIT` | TRADE 판단에서 보유 없이 지켜보는 관망 |

## 함정

- 판단 기록이 없는 주문이 있을 수 있음(의도된 동작) — 판단 기록 API가 주문을 실행하지 않아 주문 체결과 판단 기록이 따로 호출됨

## 결정

- 판단 기록 API가 주문까지 실행하는 안은 버림 — 판단 기록이 주문 도메인과 결합됨
