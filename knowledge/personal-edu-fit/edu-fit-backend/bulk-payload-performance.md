---
description: 대용량 요청 본문의 응답 조합이나 배열 전개를 구현할 때
---

# 대용량 본문 성능

## 규칙

- 업무 댓글 응답 조합(역할별 댓글 서비스 toItems)은 연결 행을 댓글 ID별 Map으로 한 번 묶어 댓글마다 첨부를 꺼냄 — 댓글마다 전체 연결 행을 filter하면 타임라인 조회가 댓글 수 × 연결 수에 비례함
- 탐구 분석 응답 조합은 후속 탐구·파일 연결·역량·활성 축·담당자를 전체에 대해 한 번씩 조회해 상위 ID별로 groupBy·Map으로 한 번만 순회해 묶음 — 상위마다 전체를 거르면 1MB 본문의 수만 개 항목에서 항목 수 제곱에 비례함
- 요청 크기에 비례하는 배열은 함수 인자로 전개(rows.push(...current), Math.max(...ids))하지 않고 flatMap·concat·반복문으로 만듦 — 1MB 본문의 최대 파일 ID 수(약 16만 5천 개)에서 RangeError(Maximum call stack size exceeded)로 500이 됨
