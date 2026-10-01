---
description: 입금 Dialog 자동화나 headless 스크린샷으로 화면을 검증할 때
---

# 브라우저 화면 검증

## 함정

- 브라우저 자동화로 입금 Dialog(DepositDialog)에 금액을 입력할 때는 `input[inputmode=decimal]`로 금액 칸을 지정함 — 통화 Select(Base UI)가 숨은 input을 Dialog 안에 먼저 렌더함
- headless Chrome으로 개발 서버 화면을 스크린샷하면 저장 뒤에도 프로세스가 끝나지 않아 직접 종료해야 함 — 개발 서버의 HMR이 종료를 막음
- headless Chrome으로 지연 응답의 로딩 상태를 찍을 때는 `--timeout`을 씀 — `--virtual-time-budget`으로는 로딩 상태가 찍히지 않음
- macOS headless Chrome으로 375px 폭 화면을 찍을 때는 `--remote-debugging-port`로 띄우고 CDP `Emulation.setDeviceMetricsOverride`(mobile: true)로 폭을 지정함 — macOS headless의 `--window-size` 폭은 약 500px 아래로 줄지 않음
