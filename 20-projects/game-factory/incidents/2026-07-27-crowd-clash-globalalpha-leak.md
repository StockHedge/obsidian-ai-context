---
게임: crowd-clash(AGY)
분류: 렌더링
심각도: High
발견일: 2026-07-27
해결일:
상태: 진단완료
aliases: [globalAlpha 누출, 튜토리얼 문구 가독성]
---

# 크라우드 클래시 — globalAlpha 누출로 튜토리얼 문구·버튼이 반투명 렌더

## 증상

실기 스크린샷에서 스테이지 1 튜토리얼 문구
`Hold and drag left/right to aim and fire!`가 하단 시안색 바 위에 겹쳐 **거의 읽히지 않는다**.
좌상단 `SKIP TUTORIAL` 버튼도 흐릿해 비활성 버튼처럼 보인다.

베타 리포트의 온보딩·가독성 감점에 직접 기여했을 항목.

## 근본원인

`src/systems/uiSystem.js:88-119`. 조준 가이드 라인을 맥동시키려고 설정한 알파가
복원되지 않은 채 이후 렌더 호출에 그대로 적용된다.

```js
const alpha = 0.4 + Math.sin(tutTimer * 4) * 0.2;   // 0.2 ~ 0.6
renderer.globalAlpha(alpha);                        // ← 여기서 설정
renderer.line(...);                                 // 조준선 (반투명 의도됨)

if (state.stage === 1) {
  renderer.text(t('tutStep1'), ...);                // ← 의도치 않게 반투명
  renderer.roundRect(skipX, ...);                   // ← 의도치 않게 반투명
  renderer.text(t('skipTutBtn'), ...);              // ← 의도치 않게 반투명
}
renderer.globalAlpha(1.0);                          // ← 복원이 여기서야 일어남
```

의도는 "조준선만 맥동"인데 실제로는 **텍스트와 버튼까지 최저 0.2 알파**로 그려진다.
문구 색 `#38bdf8`이 하단 플레이어 바의 시안 계열과 유사해, 알파까지 낮아지면 대비가
거의 사라진다.

## 수정

알파 적용 범위를 조준선 한 줄로 좁힌다.

수정 전 — `uiSystem.js:92-119`
```js
const alpha = 0.4 + Math.sin(tutTimer * 4) * 0.2;
renderer.globalAlpha(alpha);
renderer.line(cannon.x, cannon.y - 20, cannon.x, cannon.y - 120, config.theme.playerCyan, 3);

if (state.stage === 1) {
  renderer.text(t('tutStep1'), W / 2, cannon.y + 35, { color: '#38bdf8', ... });
  ...
}
renderer.globalAlpha(1.0);
```

수정 후
```js
const alpha = 0.4 + Math.sin(tutTimer * 4) * 0.2;
renderer.globalAlpha(alpha);
renderer.line(cannon.x, cannon.y - 20, cannon.x, cannon.y - 120, config.theme.playerCyan, 3);
renderer.globalAlpha(1.0);   // ← 조준선까지만 반투명. 이후 UI는 불투명.

if (state.stage === 1) {
  renderer.text(t('tutStep1'), W / 2, cannon.y + 35, { color: '#e2e8f0', ... });  // 대비 확보
  ...
}
```

문구 색도 `#38bdf8`(시안) → `#e2e8f0`(밝은 회백)로 바꿔 하단 시안 바와의 대비를 확보한다.
문구 y좌표(`cannon.y + 35`)가 플레이어 바와 겹치므로, 캐논 **위쪽**(`cannon.y - 150` 부근)
빈 공간으로 옮기는 편이 더 안전하다.

## 재발방지

`juice-coverage.node.mjs`는 이벤트에 이펙트가 **연결됐는지**만 정적 검사하므로 알파 누출 같은
렌더 상태 오염은 원리적으로 못 잡는다.

- **렌더 상태 밸런스 정적 검사**: `globalAlpha(x)` 호출 수와 `globalAlpha(1.0)` 복원 수가
  한 함수 스코프 안에서 맞는지 검사하는 린트 규칙. `save()/restore()` 패턴이 있다면 그것으로
  강제하는 편이 더 견고하다.
- 더 근본적으로는 `renderer`에 `withAlpha(a, fn)` 헬퍼를 두어 **범위를 콜백으로 한정**하면
  누출이 구조적으로 불가능해진다. 템플릿(game-standards)에 반영 대상.

## 관련

- [[2026-07-27-crowd-clash-title-phase-unreachable]] — 같은 실기 세션에서 발견된 Critical 결함
