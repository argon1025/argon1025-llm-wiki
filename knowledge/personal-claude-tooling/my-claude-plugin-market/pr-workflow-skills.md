---
description: pr-workflow의 create·review·fix 스킬이나 PR 코멘트 규약을 고칠 때
---

# pr-workflow 스킬 구성

## 적용 대상

- my-claude-plugin-market의 pr-workflow 플러그인 — create·review·fix 스킬과 references 문서, 개인판과 사내판 모두

## 규칙

### 공통

- pr-workflow는 호스트에 묶인 절차(도구, 원본 저장소 좌표·베이스, PR 조회, diff 조달, 저장소 관례, push·생성·확인, 코멘트 게시)를 host.md 1~7장에 두고, pr-protocol.md와 create·review·fix 스킬은 장 번호로 참조함
- 호스트 무관 규약(승인 게이트·금지·보고·오류)은 pr-protocol.md에 둠
- PR 호스트 도구 오류는 좌표·파라미터를 바꿔가며 재시도하지 않음 — 추측으로 성공한 조회가 엉뚱한 저장소를 가리키는 것이 실패보다 나쁨
- 앵커 지정 API 오류에만 앵커 없이 1회 재시도하고 보고함
- PR 코멘트 접두사([AI 리뷰]·[AI 코드리뷰]·[AI 반영])와 요약 상태 줄(리뷰 스냅샷: <SHA>)은 글자 하나도 바꾸지 않음 — 열려 있는 PR의 기존 코멘트와의 호환 계약이라 플러그인 이름이 바뀌어도 그대로임

### review

- review는 리뷰 룰을 2층으로 적용하며, Layer 1은 스택 감지로 고른 체크리스트(checklists), Layer 2는 세션 사전 정보에서 PR과 관련해 고른 프로젝트 문서임
- Layer 1과 Layer 2가 충돌하면 Layer 2가 이김
- 고를 Layer 2 문서가 없으면 review는 Layer 1만 쓰고 부재를 보고의 적용 룰 소스 Layer 2 칸에 없음으로만 드러내며, 다른 플러그인 미설치 알림이나 구 룰 파일(.claude/pr-review-rules.md) 이관 안내는 하지 않음
- review가 서브에이전트에 넘기는 {CHECKLIST_PATHS}와 Layer 2 {MATCHED_DOCS}는 절대 경로여야 함 — 서브에이전트는 ${CLAUDE_PLUGIN_ROOT}를 확장하지 못하고 세션에 주입된 위키 문서 목록도 물려받지 못해, 미확장 경로는 Read가 조용히 실패해 룰 없이 리뷰가 돌아감
- review의 suggestion_code는 코멘트로 게시하지 않고 보고에만 남김 — 코멘트 수정·삭제가 불가한 호스트가 있어 틀린 코드가 스레드에 남으면 되돌리기 어려움

### fix

- fix의 push 리모트는 host.md 6장이 정하며, 개인판은 소스 브랜치를 보유한 리모트(fork면 origin), 사내판은 upstream임
- fix의 push 리모트를 소스 브랜치를 보유한 리모트로 고정하지 않음 — create가 원본 저장소(upstream)에 푸시해 만든 same-repo PR을 갱신하지 못함
- fix의 승인은 둘(판정 세트 승인, push·답글 묶음 승인)임 — 기각 판정은 사람이 소유해야 하는 판단이고, 잘못된 판정 위에 쓴 커밋은 통째로 버리는 작업임

### create

- create는 PR 본문 문체를 저장소의 기존 PR 본문에서 가져오지 않음(관례 비참조) — 이 스킬이 쓴 PR이 관례로 읽히면 어긋난 문체가 굳음
