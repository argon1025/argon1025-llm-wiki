---
description: 산출물 후속 탐구 파일을 조합하거나 탐구 이력 가져오기 복사 컬럼을 정할 때
---

# 산출물 조회·가져오기 구현

## 적용 대상

- edu-fit-backend의 consulting 산출물 조회 서비스(선생·원장·학생)와 ConsultingExternalService, 산출물 값을 복사해 저장하는 analysis 탐구 이력

## 규칙

- 산출물 상세·가져오기 조회의 후속 탐구 파일 행은 후속 탐구 ID별 Map으로 한 번 묶어 조합하고 파일 정보도 한 번에 채움 — 후속 탐구마다 전체 파일 행을 filter하면 항목 수 제곱에 비례해 1MB 본문 규모(약 6만 5천 개)에서 상세 조회가 150초를 넘음

## 함정

- 탐구 이력 가져오기가 산출물 값을 복사하는 탐구 이력 컬럼(title·career·summary·keywords·후속 탐구 topic·content)은 산출물 컬럼 상한 이상이어야 함 — 작으면 복사 저장이 Data too long 500이 됨
