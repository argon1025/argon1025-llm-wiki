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
| src/shared | 둘 이상 기능이 쓰는 코드 |
| src/shared/ui | shadcn 생성물만 |
| src/shared/components | 직접 만든 공용 컴포넌트 |

- src/app의 page·layout은 컴포넌트 배치만 함
- 기능 폴더는 다른 기능 폴더를 import하지 않음
- import 방향은 app → features → shared만 허용함
- 한 기능만 쓰던 코드를 다른 기능도 쓰게 되면 그 이슈에서 shared로 승격함
- 기능 폴더는 auth·app-shell·members·inquiry-axes·consulting이며 각 화면 이슈가 자기 폴더를 만듦
- consulting 기능의 ui 세그먼트에서 학생 요청 폼 파일은 하위 폴더 없이 task-form- 접두어를 붙여 평평하게 둠 — 학생 요청 목록·상세·등록·수정과 원장 업무 관리 화면이 한 ui 세그먼트를 함께 씀
- consulting 기능은 원장 업무 화면의 학생 이름 검색·학생 상세 조회를 members의 studentQueries 대신 자체 쿼리(directorTaskQueries)로 둠 — consulting은 members를 import할 수 없음
- 원장 업무 화면의 학생 이름 검색 디바운스(300ms)는 members의 useNameSearch 대신 FilterPicker 안에서 처리함 — consulting은 members를 import할 수 없음
