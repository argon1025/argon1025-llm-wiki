---
description: shadcn 컴포넌트를 설치·수정하거나 치수·radius·hover·선택 상태를 맞출 때
---

# UI 컴포넌트 규칙

## 규칙

- 화면 커스텀은 CSS 변수 토큰(:root·.dark) → @theme inline 매핑·@utility → 사용처 className·조합 컴포넌트 → 생성물 직접 수정 순서로 하며, 앞 단계로 해결되면 다음 단계로 넘어가지 않음
- shadcn 스타일을 전환하면 components.json의 style 변경과 기존 생성물 재설치(shadcn add <name> --overwrite)만으로 끝나며 globals.css는 바뀌지 않음 — 스타일 레지스트리에 cssVars가 없음
- shadcn 생성물은 모든 화면에 같은 값을 쓰는 컨트롤 크기·radius·안쪽 여백·hover·선택 상태 규칙 범위만 직접 수정하고, 재설치(--overwrite)·업스트림 갱신 뒤 같은 규칙을 다시 적용함 — 재설치가 수정분을 덮어씀

### 컨트롤 치수

- 버튼 높이는 28·36·44·48px, 토글 높이는 36·44·48px이며 둘 다 기본은 44px
- 입력·선택 트리거는 높이 44px이며, 입력·선택·여러 줄 입력의 배경은 카드색
- 안쪽 여백은 컨트롤·팝오버·알림 20px, 카드·대화상자·시트 32px, 표 셀 세로 16px·가로 20px
- 라벨-입력 간격은 6px

### radius

- 기준 --radius는 0.5625rem

| 대상 | radius |
|---|---|
| 컨트롤·면 컨테이너 | 3xl(×2.2 = 19.8px) |
| 메뉴·선택 항목, 탭 트리거 | 2xl |
| 앱 셸 panel | 4xl |
| Badge·Checkbox·Radio·Switch·Avatar·Tooltip·Skeleton | 고유 형태 유지 |

### hover·선택 상태

- 버튼·토글 hover는 글자색 8% 겹침
- 선택 상태의 브랜드(primary) 채움은 Toggle·ToggleGroup 눌림과 Sidebar 활성 메뉴에만 적용하며 Tabs 활성 항목은 흰 바탕 세그먼트를 유지함

### 부속 요소 적용

- 치수·radius·hover·선택 상태 규칙이 이름으로 정하지 않은 의존·부속 요소는 같은 규칙을 요소 종류대로 연장 적용하며 재설치 시 함께 재적용함

| 요소 | 적용 |
|---|---|
| InputGroup | 입력 컨트롤 규칙(44px, 좌우 20px, 카드색 배경, 3xl) |
| InputGroup 안쪽 버튼 | 2xl |
| kbd | rounded-4xl 고유 형태 유지 |
| Pagination | 줄임표 size-11, 이전·다음 아이콘 쪽 pl-3!·pr-3! |
| SidebarInput | h-9, 카드색 배경 |
| Sidebar 메뉴·하위 메뉴 버튼 | 높이 유지, radius만 2xl, 활성 시 액션·배지·아이콘 글자색도 primary-foreground |
| TabsList | 방향과 무관하게 3xl |
| Calendar 루트 | rounded-3xl |

## 절차

- shadcn 컴포넌트 설치는 `yes n |`로 기존 파일 덮어쓰기를 거부하고, shadcn init·apply 재실행은 금지하며, 설치 후 globals.css가 바뀌지 않았는지 diff로 확인함

## 함정

- ToggleGroupItem의 data-[state=on]: 클래스는 동작하지 않음 — Base UI Toggle은 눌림 상태를 aria-pressed·data-pressed로만 표시하고 data-state를 붙이지 않음
- Toggle·ToggleGroup 눌림 브랜드 채움은 toggleVariants의 aria-pressed:bg-primary가 담당함

## 결정

- shadcn 스타일은 base-nova 대신 base-maia를 씀 — 알약형 버튼·입력, 원형 아이콘 버튼, 큰 카드 radius를 생성물 수정 없이 재현하며, base-nova는 사용처마다 rounded-full 등 className이 필요함
