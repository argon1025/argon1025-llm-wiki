---
description: 판단 기록의 유형·판단 코드(REGIME·TRADE·WAKE 등)를 해석할 때
type: domain
---

# 판단 유형과 판단 코드

## 용어

- 판단 유형(`type`) 국면 판단(`REGIME`): Jev가 메인 에이전트를 깨울지 여부를 판단
- 판단 유형(`type`) 거래 처리(`TRADE`): 메인 에이전트의 매매 판단
- 지갑(`pigeon_wallet`)이 에이전트 식별을 겸하며 판단 기록도 지갑별로 남음 — 에이전트 엔티티가 따로 없음

## 코드값

- 국면 판단(`REGIME`)의 판단 코드(action): `WAKE`(메인 에이전트를 깨움) · `SKIP`(깨우지 않음)
- 거래 처리(`TRADE`)의 판단 코드(action): `BUY`(매수) · `SELL`(매도) · `HOLD`(보유 유지) · `WAIT`(보유 없이 관망)
