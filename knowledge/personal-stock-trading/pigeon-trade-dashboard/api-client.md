---
description: 쿼리·뮤테이션 옵션의 재조회 주기·무효화나 조회 실패 화면을 다룰 때
---

# 조회 캐시와 실패 화면

## 규칙

- 서버 측 fetch 헬퍼와 Server Component prefetch는 쓰지 않음 — 대시보드는 프록시 없이 브라우저에서 백엔드(pigeon-trade)를 직접 호출함
- 쓰기 요청 성공 후 조회 무효화는 mutationOptions 팩토리(walletMutations)가 아니라 각 useMutation 호출부의 onSuccess에서 처리함
- 종목 목록 조회(stockQueries.list)의 재조회 주기(refetchInterval)는 queryOptions가 아니라 각 useQuery 호출부에서 지정함 — 마켓 목록·종목 상세·지갑 상세 세 곳이 공유함
- 폴링 화면은 isError가 아니라 데이터 유무(data === undefined)로 오류 화면을 가름 — TanStack Query v5는 데이터가 있는 상태에서 재조회가 실패해도 isError가 true가 됨

## 함정

- 대시보드 QueryClient는 기본 retry 3회를 유지해 조회 실패·404 WALLET_NOT_FOUND 오류 문구가 약 7초 뒤에 표시됨(의도된 동작)
- 공용 toLoadState를 쓰는 종목 차트(30초 폴링)와 지갑 화면은 데이터가 있어도 재조회가 한 번 실패하면 오류 블록으로 바뀜 — toLoadState가 데이터 유무보다 isError를 먼저 봄
