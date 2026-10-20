---
description: 프론트엔드에 새 파일·기능 폴더를 만들거나 shared로 옮길 때
---

# 프론트엔드 폴더 구조

## 적용 대상

- edu-fit-frontend의 src 아래 코드 배치 (경량 기능 폴더 구조)

## 규칙

| 위치 | 두는 코드 |
|---|---|
| src/app | 라우팅 파일·루트 레이아웃·globals.css만 (개발 전용 dev/**·mock/**는 예외) |
| src/features/{기능} | 한 기능 전용 코드를 ui·api·model·lib 세그먼트로 나눠 둠 |
| src/shared | 둘 이상 기능이 쓴다고 이슈 스펙으로 확정된 코드 |
| src/shared/ui | shadcn 생성물만 |
| src/shared/components | 직접 만든 공용 컴포넌트 |

- src/app의 page·layout은 컴포넌트 배치만 함
- 기능 폴더는 다른 기능 폴더를 import하지 않음
- import 방향은 app → features → shared만 허용함
- 다른 기능의 사용이 이슈 스펙으로 확정된 코드는 처음 만드는 이슈에서 shared에 둠 — 기능 폴더는 다른 기능 폴더를 import하지 않음
- 다른 기능의 사용이 이슈 스펙으로 확정되지 않은 코드는 기능 폴더에 두고, 다른 기능이 쓰게 되는 이슈에서 shared로 승격함
- 기능 폴더는 auth·app-shell·members·inquiry-axes·consulting이며 각 화면 이슈가 자기 폴더를 만듦
