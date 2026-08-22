---
type: incident
schema_version: 1
project: game-factory
component: coop-defense / matgo-pocket (Play 등재정보)
category: 유통
severity: Medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-22
resolved: 2026-08-22
verified: 2026-08-22
agents: [claude-code]
source_repo: StockHedge/various-game
source_commit: 69727bf
aliases: [등재정보 노후화, 스크린샷 0장 회귀, 로케일 추가 사고]
tags: [store-listing, play-console, release-pipeline, definition-of-done, aso]
---

# 등재정보가 릴리스 파이프라인 밖에 있어 앱과 따로 늙었다

## 무슨 일이 있었나

Play 배포 감사(P0) 도중 coop-defense 의 라이브 등재정보를 실제 게임과 대조했다.

- 등재는 `맵 3 / 유닛 12` 라고 적혀 있었다. 실제는 `MAPS 7 / UNITS 25 / ENEMIES 15 / WAVES 25`
  — **2배 이상 과소 서술**이다.
- 등재에 "AI 파트너와 자원을 공유한다" 는 취지의 서술이 있었다. 코드는 반대다
  (`state.js`: gold 는 플레이어 지갑이고 AI 는 건드리지 않는다). **없는 메커닉을 광고**한 셈이다.
- 로케일은 `ko-KR` 하나뿐이었다.

원인은 단순하다. **버전 코드·AAB 는 `play-release.mjs` 가 릴리스마다 갱신하지만,
등재정보는 사람이 최초 1회 쓰고 그 뒤 아무도 손대지 않는 경로에 있었다.** 게임은
계속 자라는데 그 설명은 첫 출시 시점에 고정돼 있다. 릴리스의 완료 정의(DoD)에
등재정보가 들어 있지 않았다.

## 2차 사고 — 고치다가 스크린샷을 0장으로 만들었다

등재를 5개 로케일로 확장하면서 en/ja/es 로케일을 새로 만들었다. 그런데 그 3개
로케일의 스크린샷이 **0장**이 됐다.

`play-release.mjs` 는 `store-assets/graphics/screenshots/` 규약 경로에서 이미지를 읽는다.
coop-defense 만 그 디렉터리가 없는 비표준 배치였다. 기존 `ko-KR` 은 과거에 손으로
올려놔서 살아 있었고, 새 로케일은 읽을 소스가 없어 빈 채로 생성됐다. **로케일 추가가
곧 "그 로케일에 스크린샷 없는 등재를 신설"** 이었던 것이다.

Play 는 스크린샷 없는 로케일도 그대로 받는다 — 실패하지 않으므로 도구 성공 응답만
보면 알 수 없다. 자산 배치를 규약대로 정규화하고 재업로드해 복구했다 (`69727bf`).

## 근본 원인

1. **등재정보가 릴리스 파이프라인 밖에 있다.** 코드/빌드는 자동, 등재는 수동 1회.
   두 개가 같은 릴리스 단위로 묶이지 않으면 시간이 갈수록 벌어진다.
2. **자산 배치의 비표준을 도구가 조용히 흡수했다.** 소스 없음을 에러가 아니라
   "올릴 게 없음" 으로 처리하면, 규약 이탈이 결함으로 드러나지 않고 축적된다.

## 재발 방지

- 릴리스 DoD 에 **"등재정보가 이번 빌드의 콘텐츠 규모와 일치하는가"** 를 넣는다.
  숫자(맵/유닛/스테이지 수)는 코드의 상수와 대조 가능한 형태로 쓴다.
- 등재 서술은 **코드로 검증 가능한 주장만** 쓴다. 검증할 수 없는 매력 서술은
  메커닉 주장으로 쓰지 않는다.
- 로케일 신설 시 **스크린샷 장수를 업로드 후 재조회로 확인**한다. 도구의 성공 응답은
  검증이 아니다 (같은 계열: [[2026-08-21-verification-bypass-blind-spot]]).
- 자산 배치가 규약과 다르면 **먼저 정규화한 뒤** 작업한다. 예외를 안고 진행하면
  그 예외가 다음 사고의 원인이 된다.

## 부수 슬립 — `git add -- <디렉터리>`

이 복구 커밋(`69727bf`)에 타 세션 소유 변경
(`mobile-claude-code-1/android/app/build.gradle`, vc3→4)이 섞였다. 디렉터리 단위
스테이징이 원인이다. 되돌리면 git 이 라이브 vc4 와 어긋나 더 나빠지므로 그대로 두고
기록만 남겼다. **모노레포에서 `git add` 는 파일 단위로 한다** — 규약은
`various-game/README.md` 와 `game-factory/docs/ai/NOW.md` 에 반영했다.

## 관련

- [[management-api-blind-spot-observe-public-surface]] — 같은 P0 감사에서 나온 패턴
- `various-game/game-factory/docs/growth/P0-distribution-audit-2026-08-21.md`
