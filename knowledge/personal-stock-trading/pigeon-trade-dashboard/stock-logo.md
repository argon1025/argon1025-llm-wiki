---
description: 종목 로고를 표시하거나 로고 소스를 바꿀 때
---

# 종목 로고

## 규칙

- 종목 로고(StockLogo)를 보여주는 모든 페이지에 elbstream.com 출처 링크(LogoAttribution)를 12pt 이상으로 표시함 — Elbstream 무료 이용 조건임

## 함정

- 출처 링크(LogoAttribution)의 `text-[12pt]`를 12px 캡션 크기로 줄이면 Elbstream 무료 이용 조건을 어김 — 조건 단위가 pt라 12pt는 CSS 16px임
- 국내 종목은 Elbstream 로고를 ISIN 조회로만 받음 — 6자리 숫자 심볼 조회는 404를 반환함
- Elbstream에 로고가 없는 종목(SK스퀘어의 ISIN·심볼, SOXS의 ISIN)이 마켓 목록 브라우저 콘솔에 `Failed to load resource` 404를 남기는 것은 정상임 — `<img onError>`가 404를 받아 다음 소스로 넘어가는 대체 처리라 제거 대상이 아님

## 결정

- 토스 정적 CDN 종목 아이콘은 로고 소스로 쓰지 않음 — 등록 종목을 모두 제공하지만 공개 API가 아닌 토스 앱 내부 자산이라 이용 조건·지속성이 보장되지 않음
- assets.parqet.com 로고는 누락 보완용 대체 소스로 쓰지 않음 — Elbstream과 같은 데이터 소스라 누락을 채우지 못함
