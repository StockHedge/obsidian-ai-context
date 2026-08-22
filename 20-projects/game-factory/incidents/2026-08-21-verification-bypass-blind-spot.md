---
type: incident
schema_version: 1
project: game-factory
component: crowd-clash(AGY)
category: 검증
severity: Medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-21
resolved: 2026-08-21
verified: 2026-08-21
agents: [claude-code, cursor-agent]
source_repo: StockHedge/various-game
source_commit: c6a5e4f
aliases: [검증 우회 사각지대, pressed 래치 유출, CDP 강제 전환]
tags: [verification, blind-spot, input-latch, phase-gate, code-review]
---

# 검증 우회가 만든 사각지대 — 에뮬 실검증을 통과한 변경에 회귀가 남아 있었다

## 요약

다른 AI 세션(Cursor Agent)이 게임 5종을 픽스하고 **에뮬레이터 실설치·실플레이**로
검증한 뒤 리포트를 남겼다. 층1 게이트(`npm test`) 5종 EXIT=0, 증거 스크린샷 55장.
독립 재검증에서 그 변경이 **직접 도입한 회귀 1건**을 발견했다. 검증이 부실해서가
아니라, 검증 도구의 한계를 우회한 방식이 정확히 그 결함을 가렸다.

## 증상

crowd-clash에서 게임 시작 버튼을 누르면 PLAYING 첫 프레임에 유닛이 1발 오발되고
캐논이 시작 버튼 좌표로 순간이동한다. 그리고 **스테이지 1 유예 구간이 사라진다**.

## 근본원인 (확인됨)

리뷰가 추가한 짧은 탭 래치(`pointer.pressed`)의 소진이 `cannonSystem.update`의
phase 게이트 **뒤**에 있었다.

```js
update(dt) {
  if (state.phase !== 'PLAYING') return;   // TITLE/PAUSED 에서 여기서 반환
  ...
  pointer.pressed = false;                 // 그래서 실행되지 않는다
}
```

게임 시작 버튼은 `uiSystem`(업데이트 순서 **마지막**)이 `phase = PLAYING`으로
바꾼다. 타이틀 탭으로 켜진 `pressed = true`가 해제되지 않은 채 다음 프레임으로
넘어간다. 일시정지 해제 경로도 같다.

영향이 오발에 그치지 않는 이유: `hasPlayerActed`는 스테이지 1 유예 게이트이고
`defenseSystem`·`enemySystem`·`crowdSystem` 3개가 이 값을 본다. 게임을 시작한 그
탭 하나가 유예를 통째로 날린다.

## 왜 에뮬 실검증이 못 잡았나 — 이 사건의 핵심

리포트 §4.2에 그대로 적혀 있다. crowd-clash는 **레터박스 캔버스**(약 412×732) 밖의
`adb tap`이 no-op이라 인게임 진입이 안 됐고, 그래서 **CDP로 `phase=PLAYING`을
강제**한 뒤 검증을 이어갔다.

그 우회가 정확히 버그를 유발하는 **타이틀→플레이 탭 경로를 건너뛴다.** 입력 경로를
우회해 상태를 주입하면, 그 입력 경로에 있는 결함은 정의상 검출되지 않는다.
우회는 검증을 진행시켰지만 동시에 검증 대상 일부를 삭제했다.

층1 게이트도 못 잡는다 — `idle-survival.node.mjs`는 무입력 생존만 본다.

## 증거와 재현 방법

수정 전/후를 동일 스텁 ctx로 대조 실행(타이틀 탭 → PLAYING 1프레임):

```
수정 전 | 타이틀 후 pressed=true  | PLAYING 1F: hasPlayerActed=true  targetX=300 오발
수정 후 | 타이틀 후 pressed=false | PLAYING 1F: hasPlayerActed=false targetX=270 정상
```

## 수정

래치 소진을 phase 게이트 **앞**으로 옮겨 페이즈와 무관하게 매 프레임 소진한다.
`crowd-clash(AGY)/src/systems/cannonSystem.js`, `+7 / −2`.

## 검증 결과

5게임 `npm test` EXIT=0 재실행. 위 대조 실행으로 회귀 소거 실증. 커밋 `c6a5e4f`.

## 재발방지

- 입력 래치는 **소비자의 조기 반환 뒤에 두지 않는다.** 소진은 게이트 앞에서.
- 자동화 편의를 위해 추가한 코드 경로(짧은 탭 래치)도 실제 게임 상태 전이를
  거친다 — 테스트 전용이라고 가정하지 않는다.

## 다른 프로젝트에도 적용할 규칙

**검증을 우회했다면 무엇을 못 보게 됐는지 함께 기록한다.** 도구 한계로 우회할 때
(좌표가 안 맞아 CDP로 상태 주입, 광고 클릭 금지로 실패 경로 미실연, 로그인 스텁 등)
그 우회는 검증 범위에서 해당 경로를 **삭제**한 것이다. 리포트에 "우회함"만 적으면
읽는 쪽은 통과로 읽는다. "우회 → 따라서 X 경로는 미검증"까지 적어야 한다.

이 리포트는 광고 실패 경로도 같은 이유로 미실연이라고 적었다(§6-6). 그쪽은
정직하게 남겼고 이쪽은 남기지 않은 차이가 결과를 갈랐다.

**부수 확인:** 자기평가 신뢰도 문제이기도 하다. 리포트 §8이 스스로 "층2 자가 채점을
층1 대신 쓰지 말 것"이라 적었는데, 같은 원리가 **에뮬 실검증 자기보고**에도 적용된다.

## 관련 커밋과 문서

- `various-game/game-factory/docs/reviews/2026-08-20-monorepo-code-review.md`
  (§3.6 · §4.3 에 이 건 반영, 2026-08-21 정정)
- 커밋 `c6a5e4f`
- [[2026-08-17-various-game-monorepo-consolidation]]
