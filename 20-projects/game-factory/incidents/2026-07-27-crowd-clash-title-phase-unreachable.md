---
게임: crowd-clash(AGY)
분류: 게임로직
심각도: Critical
발견일: 2026-07-27
해결일:
상태: 진단완료
aliases: [타이틀 화면 도달 불가, crowd-clash 즉시 자동패배]
---

# 크라우드 클래시 — 타이틀 화면이 도달 불가능하고 방치 시 즉시 패배

## 증상

안드로이드 실기(에뮬레이터 `coop_td`, API 36)에 debug APK를 설치해 관찰한 결과:

1. 앱을 실행하면 **타이틀 화면이 전혀 표시되지 않고** 곧바로 스테이지 1 플레이가 시작된다.
2. 플레이어가 아무 입력도 하지 않으면 **약 10초 만에 `DEFEAT...`** 결과 화면이 뜬다
   (재현 2회, 둘 다 SCORE 40에서 패배).
3. 반대로 조준 드래그를 넣어주면 정상적으로 `VICTORY!` + `Earned Coins: +138`까지 완주한다.

베타테스트에서 이 증상이 **"메인화면 입력 무반응"**으로 보고됐다. 테스터(AI 페르소나)가
화면을 인식하고 판단하는 수 초 사이에 이미 패배 화면으로 전환돼, 조작이 먹지 않는 것처럼
보인 것이다. 베타 26.5/100 2회 미달 → `escalated(beta_failure)`의 실제 원인.

## 근본원인

`src/core/state.js:12`는 초기 phase를 올바르게 `PHASE.TITLE`로 설정한다.
`src/systems/uiSystem.js:128-185`에도 타이틀 화면이 **완전히 구현돼 있다**(브랜드 로고,
`MULTIPLY & DEMOLISH` 카피, 탭하면 `state.phase = PHASE.PLAYING` 전환).

그런데 `src/systems/stageSystem.js:198`이 시스템 생성 시점에 무조건 `loadStage(1)`을
호출하고, `loadStage`는 `stageSystem.js:9`에서 `state.phase = 'PLAYING'`으로 덮어쓴다.

```
createGame() → createStageSystem(ctx) → loadStage(1) → state.phase = 'PLAYING'
```

시스템 등록이 곧 게임 시작이 되어버려, **`PHASE.TITLE`과 그 렌더링 분기 전체가 도달 불가능한
죽은 코드**가 됐다. 타이틀이 없으니 플레이어는 준비할 틈 없이 진행 중인 스테이지에 던져지고,
화면을 파악하는 동안 `playerBase.hp`(100)가 적 타워 사격으로 소진돼 패배한다.

`main.js:213`의 `getSpeed`가 `phase === 'TITLE'`일 때 0을 반환하는 분기 역시 같은 이유로
한 번도 실행되지 않는다.

## 수정

`loadStage`는 스테이지 데이터(게이트·타워·초기 유닛)를 만들어야 타이틀 배경에 렌더할 것이
생기므로 호출 자체는 유지하되, **부팅 경로에서만 phase를 TITLE로 되돌린다.**

수정 전 — `src/systems/stageSystem.js:197-198`
```js
// Load initial stage 1
loadStage(1);
```

수정 후
```js
// Load initial stage 1 for the title backdrop — the TITLE screen (uiSystem)
// owns the transition to PLAYING, so booting must not auto-start the run.
loadStage(1);
state.phase = 'TITLE';
```

`restartStage()`/`nextStage()`는 `requestStageLoad` 이벤트를 거치므로 영향이 없다
(그 경로에서는 `PLAYING`이 올바른 결과다).

추가로 필요한 보완 — 타이틀을 복원해도 튜토리얼을 읽는 동안 패배할 여지는 남는다:

- 스테이지 1 한정 **유예 구간**: 첫 플레이어 입력 전까지 또는 시작 후 3초간 적 타워 사격과
  적 유닛 전진을 정지시킨다(`defenseSystem`/`enemySystem`의 update 초입에서 gate).
- `uiSystem.js:116`의 SKIP TUTORIAL이 `requestStageLoad({ stage: 2 })`로 **스테이지를
  건너뛴다**. 튜토리얼 스킵이 난이도 상승(스테이지 2는 `spiked_roller` 추가)을 부르는 것은
  의도와 반대이므로, 스테이지 1을 유지한 채 튜토리얼 오버레이만 끄도록 바꾼다.

## 재발방지

이 결함은 게이트 6종(determinism/no-sim-random/dom/sw-cache/content-volume/juice-coverage)을
전부 통과했고 독립 감사 4.43 PASS까지 받았다. 통과한 이유가 핵심이다:

- `dom.node.mjs`는 jsdom에서 프레임을 돌리지만 **"부팅 직후 어떤 phase인가"를 검사하지 않는다.**
- Playwright 감사는 스크립트가 0초 만에 정확한 입력을 넣으므로 절대 패배하지 않는다.

신설 제안 — **`idle-survival.node.mjs` 게이트** (플랫폼 무관, 초저비용):
1. 헤드리스로 boot 후 첫 프레임의 `state.phase`가 `TITLE`인지 단언한다.
   (타이틀 없이 시작하는 게임은 이 시점에 실패)
2. 타이틀을 통과시킨 뒤 **입력을 일절 넣지 않고** 고정 dt로 N초(기본 20초) 분량을 돌려
   `phase`가 `STAGE_FAIL`이 되지 않는지 단언한다.

"에뮬레이터 스모크 게이트"로는 이 갭을 못 잡는다 — 스모크도 스크립트 입력이기 때문이다.
잡아야 하는 것은 플랫폼 차이가 아니라 **무입력 생존 시간**이다.

## 관련

- [[2026-07-27-webview-misdiagnosis]] — 이 결함이 "WebView 전용 결함"으로 오진된 경위
- [[2026-07-27-crowd-clash-globalalpha-leak]] — 같은 실기 세션에서 발견된 렌더링 결함
