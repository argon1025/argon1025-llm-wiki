---
description: 세션 시작 훅의 위키 주입 동작을 고치거나 점검할 때
---

# 세션 시작 훅

## 규칙

- 훅(session_start.py)은 origin 판정을 위키 동기화보다 먼저 함 — git 밖이거나 origin 없는 폴더에서 주입할 것이 없는데도 최대 3초를 기다리지 않게 하기 위함
- 주입은 블록마다 텍스트 생성기(generate_wiki_rules·generate_repository_map·generate_document_list)를 따로 두고 훅이 순서대로 실행해 합침 — 나중에 수정·제외하기 쉽게 하기 위함
- 레포 지도 표식(현재 레포가 의존·현재 레포에 의존)은 deps.json의 그룹 키와 to만 읽어 desc 없이 계산함 — 구형 kind·contracts 간선에서도 동작함
- 주입의 위키 수정 금지(generate_wiki_rules)는 임시 clone에도 적용되어 일반 세션 에이전트는 임시 clone으로도 위키를 고치지 않음
- 위키 문서는 하위 폴더나 description 외 frontmatter 키에 기댈 수 없음 — 주입의 문서 목록(generate_document_list)이 도메인 루트와 레포 폴더의 *.md만 한 단계 glob하고 description 한 줄만 읽음
- 문서 목록(generate_document_list)은 description 양끝의 큰따옴표·작은따옴표를 지워 보여 줌 — @로 시작하는 등 YAML상 따옴표로 감싸야 하는 description이 세션 목록에 따옴표째 보이지 않게 하기 위함

## 함정

- 위키 동기화(sync_wiki)가 3초를 넘기면 정리가 뒤에서 마저 도는 동안 이번 주입이 반쯤 바뀐 위키 트리를 읽을 수 있음(의도된 동작) — 다음 세션에서 바로잡힘
- git 밖이거나 origin 없는 폴더의 세션에서는 위키 사본도 최신화되지 않음(정상)
