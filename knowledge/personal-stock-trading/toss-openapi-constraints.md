---
description: 토스증권 Open API로 시세를 받거나 응답을 처리할 때
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
| 캔들 조회 | 요청당 200개 | 배치 백필 |
| 미국 시세 | 일부 거래소 기준, 1분봉 이상값 가능 | 지표 계산 전 이상값 처리 |
| 데이터 이용 | 본인 매매 목적만 | 대시보드 본인 전용 |

## 명세

- 정본 명세는 https://openapi.tossinvest.com/openapi-docs/latest/openapi.json — 개발자센터 llms.txt가 이 경로를 정본으로 지정함

## 응답 형식

- 성공 응답은 `{ result }` envelope, 실패는 `{ error: { requestId, code, message, data? } }`
- 토큰 발급(POST /oauth2/token)만 예외 — envelope 없는 OAuth2 표준 형식(access_token·token_type·expires_in)이고, 실패는 `{ error: 'invalid_client', error_description }`, 요청 본문은 application/x-www-form-urlencoded

## nullable 필드

- 호가·현재가 timestamp, 상하한가 upperLimitPrice·lowerLimitPrice, 캔들 nextBefore는 명세 required에 없어 응답에서 빠질 수 있음 — 소비 측은 undefined와 null을 모두 처리해야 함

## 캔들 조회 상세

- 캔들 timestamp는 1분봉이면 봉 종료 시각([timestamp - 1분, timestamp) 구간 집계), 일봉이면 해당 거래일 현지 자정
- 캔들 목록은 최신순이며 다음 페이지는 nextBefore를 before에 그대로 전달
- before 파라미터의 타임존 +는 %2B로 인코딩해야 함 — URLSearchParams는 자동 처리함
