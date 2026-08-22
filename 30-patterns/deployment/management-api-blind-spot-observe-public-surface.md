---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-08-22
updated: 2026-08-22
source_projects: [game-factory]
agents: [claude-code]
tags: [deployment, observability, store, api-limits, verification]
---

# 관리 API가 노출하지 않는 상태는 공개 표면에서 관측한다

## 문제

배포 상태를 자동으로 확인할 때 관리 API(Publishing/Admin API)만 본다. 그런데 **관리
API가 모든 상태를 노출하지는 않는다.** 노출하지 않는 항목은 "이상 없음"이 아니라
"관측 불가"인데, 자동화는 둘을 구분하지 못하고 조용히 통과시킨다.

2026-08-22 실증(game-factory): Google Play Publishing API v3 로 앱 6종을 감사했다.
트랙·릴리스·등재 로케일은 전부 보였다. 그런데 **앱 단위 국가 배포 설정은 보이지 않았다**
— `edits.countryavailability` 는 6종 전부 404, `track.releases[].countryTargeting` 은
전부 `null`(= "배포 계층에 위임"). API 만으로는 영원히 볼 수 없는 값이다.

그 값이 중요했다. 한국어 화투 게임 하나가 IARC 도박 정책으로 **타깃 시장인 한국에서
배포 제외** 상태였다. 스토어 방문자 0을 "유통 cold-start" 로 진단하고 트래픽 예산을
태울 뻔했다 — 실제로는 그 시장에서 페이지 자체가 존재하지 않았다.

## 재사용 가능한 원칙

**제품이 사용자에게 보이는 그 표면에서 직접 관측한다.** 관리 API는 내가 무엇을
설정했는지 말해주고, 공개 표면은 사용자가 무엇을 보는지 말해준다. 둘은 다르다.

Play 의 경우 공개 상세 페이지에 국가 코드를 붙이면 결정적 신호가 나온다:

```
GET https://play.google.com/store/apps/details?id=<pkg>&gl=<국가>
  → HTTP 200 = 그 국가에서 접근 가능
  → HTTP 404 = 미배포
```

같은 방식으로 검색 노출도 확인할 수 있다(`/store/search?q=…&gl=…&hl=…` 의 서버 렌더
HTML 에 패키지명이 있는지). **단 검색은 개인화·지역·기기 영향을 받으므로 부재는
정황이지 증명이 아니다** — 이 한계를 함께 기록해야 한다.

## 적용 조건

- 관리 API 응답에 기대한 필드가 없거나 404 를 준다 (= 계약이 아니라 공백)
- 그 상태가 사용자 경험을 좌우한다 (배포 국가, 가시성, 가격, 등급)
- 공개 표면이 존재하고 인증 없이 관측 가능하다

## 적용하지 말아야 할 조건

- 관리 API 가 그 값을 제대로 노출한다 — 굳이 스크래핑하지 않는다
- 공개 표면이 강하게 개인화된다 — 신호로 못 쓴다. 쓰더라도 "정황" 이라고 못박는다
- 대량·고빈도 폴링 — 공개 엔드포인트를 몰아치지 않는다. 감사 주기로만 쓴다

## 확인 절차

1. 관리 API 응답에서 **없는 필드**를 목록화한다. "없다"를 "정상"으로 읽지 않는다
2. 각 공백에 대해 공개 표면에서 관측 가능한지 확인한다
3. 관측 결과를 관리 API 결과와 **같은 리포트에** 넣되 출처를 구분한다
4. 관측 불가로 남는 항목은 **수동 검증 체크리스트로 명시 이관**한다 — 조용히 빠뜨리지 않는다

## 근거 사건

- `various-game/game-factory/docs/growth/P0-distribution-audit-2026-08-21.md`
- 도구: `game-factory/tools/play-audit.mjs` (읽기 전용 — `edits.commit` 경로 없음)
- 커밋 `d2bfb9a`
- 관련: [[2026-08-21-verification-bypass-blind-spot]] — 검증 수단이 만드는 사각지대라는
  같은 계열. 그쪽은 *우회*가, 이쪽은 *API 공백*이 원인이다.
