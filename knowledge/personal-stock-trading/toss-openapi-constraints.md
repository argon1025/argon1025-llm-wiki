---
description: 토스증권 Open API로 시세를 받거나 리플레이 구간을 정할 때
type: external
---

# 토스증권 Open API 제약

## 적용 대상

- MVP는 토스를 시세 공급원으로만 사용하며 아래 제약만 영향을 줌

## 제약

| 항목 | 제약 | 대응 |
| --- | --- | --- |
| 토큰 | 클라이언트당 유효 토큰 1개 | 백엔드가 독점 |
| 허용 IP | 호출 IP 변경 시 차단 | 홈서버 운영이라 고정 공인 IP 또는 중계 필요, 확보 방법은 아직 정해지지 않음 |
| 과거 데이터 | 국내 2022-11-23, 미국 2021-11-30 이후 | 리플레이 구간은 이 범위 안 |
| 미국 시세 | 일부 거래소 기준, 1분봉 이상값 가능 | 지표 계산 전 이상값 처리 |
| 데이터 이용 | 본인 매매 목적만 | 대시보드 본인 전용 |

## API 형식

- 정본 명세는 `https://openapi.tossinvest.com/openapi-docs/latest/openapi.json` — 개발자센터 `llms.txt`가 이 경로를 정본으로 지정함
- 4xx·5xx 오류 본문은 `{ error: { requestId, code, message, data? } }` 형식이나 토큰 발급(`POST /oauth2/token`) 실패만 `{ error: 'invalid_client', error_description }` 형식
- 통화 코드(`Currency`)는 `KRW`·`USD`이나 명세가 알 수 없는 enum 값 허용을 요구함
