---
type: incident
schema_version: 1
project: game-factory
component: alchemy-bounce-master
status: resolved
root_cause_status: confirmed
게임: alchemy-bounce-master(AGY)
분류: 코드버그
심각도: 보통
발견일: 2026-07-23
해결일: 2026-07-23
상태: 해결
aliases: [스테이지 클리어 플레이스홀더 미치환]
태그: [agy-lane, i18n, ui, 감사발견]
---

# 스테이지 클리어 모달 {gold}/{essence} 플레이스홀더 미치환

**발견 경로**: AGY 독립 감사 라운드 1 (로컬 Playwright 실플레이) — **파이프라인이 정상
작동해 잡아낸 첫 결함**. 실캡처 `screenshots/07-stage-result.png`에 원문 그대로 노출:
"획득 골드: +{gold}🪙 / 획득 에센스: +{essence}🧪" (점수 2050은 정상 치환).

## 근본원인 (추정)
`src/data/strings.js`의 `clearGoldMsg`/`clearEssenceMsg`는 `{gold}`/`{essence}` params를
받는 템플릿인데, `src/main.js:343` `bus.on('stageClear')` 핸들러가 `t(key)` 호출 시
params 전달 누락 → i18n 치환 미실행.

## 수정
**부분 수정(미완)** — dev 커밋 ed8092c가 `t()` params 전달은 고쳤으나(치환 자체는 동작),
**진짜 원인은 emit payload**: `stageSystem.js:89` `bus.emit('stageClear', {stageIndex,
score, stars})`에 gold/essence 필드가 아예 없다 → `gold || 0` 폴백 → 모달에 **+0/+0**
표시 (Claude 레인 실플레이 재현으로 확정, 2026-07-23. 실제 보너스는 state에는 가산됨 —
표시만 0). 올바른 수정 = emit payload에 stageGoldBonus/stageEssenceBonus 포함.

## 최종 해결 (2026-07-23 저녁, 커밋 7a58b69 — Claude 레인 검수 확인)
`stageSystem.js` emit payload에 `gold: stageGoldBonus, essence: stageEssenceBonus` 추가 +
CONTRACT.md 이벤트 정의 갱신. 검수 증거: `stage1-clear-verified.png`(+50/+30 = 공식값),
베타 r2 `beta-core-stage3-clear.png`(+150/+90 = 스테이지3 공식값, 점수 8080 별개 실런).
베타 라운드 2는 캡처 인용 규칙을 준수해 82.0/100 PASS — 판정 유효.

## 3차 무결성 사건 (베타 세션)
베타 로그가 innerText로 "+150🪙/+80🧪"를 읽었다고 주장하며 "CRITICAL 재검증 PASS"
결론을 냈으나, **자신이 첨부한 캡처는 +0/+0** — 코드 공식((stage)*50/*30)과도, 어떤
실행 경로와도 일치하지 않는 창작 수치. 베타 PASS(85.33) 판정 무효. 온보딩 §7에
"캡처 > 로그, 불일치 = 리포트 무효" 조항 추가로 대응.

## 재발방지
- i18n 템플릿 키는 params 필수 여부를 validateData 수준에서 검사하는 게이트 후보
  (game-standards 개선 제안 — PROPOSALS.md 대상).

## 관련
- audit-result.json directives[0] (CRITICAL) · [[2026-07-23-agy-black-canvas-fake-screenshots]]
