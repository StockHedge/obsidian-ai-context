---
type: milestone
schema_version: 1
project: game-factory
component: various-game (포트폴리오 전체)
status: monitoring
verified: 2026-08-22
agents: [claude-code]
source_repo: StockHedge/various-game
source_commit: 5c8082e
aliases: [유통 우선 전환, 배포 감사 P0, 계측 도입 P1]
tags: [growth, distribution, analytics, play-store, pivot]
---

# 국면 전환 — 신규 게임 생산에서 유통·계측으로

## 무엇이 바뀌었나

게임공장은 지금까지 **생산 속도**를 최적화해 왔다. 게임 8종을 만들었고 6종을
Play 에 올렸다. 그런데 Store Visitor 가 사실상 0이었다.

이 지표 앞에서 7번째 게임은 정보 가치가 거의 없다. 게임성 가설을 검증하려면
플레이어가 있어야 하는데, 플레이어가 0이면 어떤 게임을 만들어도 결과가 같다.
**병목은 게임성이 아니라 유통이다.** 2026-08-22 부로 신규 게임 개발을 멈추고
유통과 측정 인프라로 자원을 옮겼다.

## P0 — 배포 정상성 감사에서 나온 것

읽기 전용 도구 `game-factory/tools/play-audit.mjs` 를 만들어 6종을 감사했다.
**커밋 경로를 의도적으로 넣지 않았다** — 상태를 보는 도구가 상태를 바꿀 수 있으면
감사가 아니다.

두 건이 나왔다. 둘 다 "게임이 안 팔린다" 가 아니라 **"팔 수 있는 상태가 아니었다"** 다.

- **matgo-pocket 이 한국에서 404.** IARC 도박 시뮬레이션 등급으로 타깃 시장에서
  배포 제외돼 있었다. 한국어 화투 게임인데 한국에서 페이지가 없다.
- **order-pop 이 전 세계 404.** API 는 `production/completed/vc3` 라고 보고하는데
  8개국 어디서도 상세 페이지가 열리지 않는다. 심사 미통과 상태다.

관리 API 만 봤으면 둘 다 못 봤다 — 이 관측 방법 자체가 재사용 자산이라
[[management-api-blind-spot-observe-public-surface]] 로 따로 승격했다.

## 대응

- **matgo-pocket = A안(비한국 시장 재포지셔닝).** 한국을 되찾으려면 GRAC 직접 심의가
  필요해 이번 사이클 밖이다. 대신 일본·영어권·스페인어권으로 방향을 돌렸다.
  하드코딩 한국어 4곳 i18n 수정 → 게이트 → v1.0.1(vc3) 릴리스 → **릴리스된 빌드로**
  로케일별 스크린샷 재촬영 → 등재 재작성. 순서를 이렇게 잡은 이유는 스토어 스크린샷이
  출시되지 않은 빌드를 보여주면 안 되기 때문이다.
  등재 제목은 앱 이름 머리를 유지했다(`Pocket Matgo: Hanafuda Go-Stop`) —
  2026-07-29 등재-앱 불일치 반려 전례를 재현하지 않기 위해서다.
- **coop-defense** 등재정보가 게임을 2배 이상 과소 서술하고 없는 메커닉을 광고하고
  있었다. 별도 사건으로 기록: [[2026-08-22-store-listing-outside-release-pipeline]]
- **order-pop 은 트래픽 투입 금지.** 심사부터 통과시켜야 한다.

## P1 — 계측 스키마와 2게임 적용

스키마 `game-standards/docs/70-analytics.md`, 구현
`game-standards/template/src/services/analytics.js`.

설계 원칙 하나가 전부다: **게임 시스템을 건드리지 않는다.** 게임마다 코드에 이벤트를
심으면 게임 수만큼 유지보수가 늘고 게임마다 다른 이름이 생긴다. 대신 이미 존재하는
이벤트 버스를 구독해 표준 이벤트로 **옮긴다**. 게임당 추가되는 것은 매핑 테이블 1개와
`main.js` 3줄이다.

진행 단위가 게임마다 다른 것(스테이지/목표/라운드/웨이브)은 `unit_type` 으로 정규화해
한 통에서 비교 가능하게 했다. block-clear 처럼 목표를 무작위 추첨하는 게임은 순번을
줄 수 없어 어댑터가 판 내 일련번호를 부여하고 어떤 목표였는지는 `level_id` 로 남긴다.

게이트 `tests/analytics.node.mjs` 가 **매핑↔버스 정합**을 검사한다. 버스 이벤트 이름이
바뀌면 계측이 조용히 멈추는 대신 테스트가 깨진다.

적용: tangle-out, block-clear. 아직 `consoleTransport` 라 아무 데도 보내지 않는다 —
백엔드 선정이 Phase 2 진입 조건이다.

## 남은 것

`various-game/game-factory/docs/ai/NOW.md` 가 현재 상태의 단일 출처다.
심사 결과 확인(matgo·alchemy·coop-defense·order-pop) → 계측 백엔드 선정 →
투입 가능한 게임에 스모크 트래픽 30~50 Store Visitor.
