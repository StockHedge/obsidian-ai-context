---
type: incident
schema_version: 1
project: game-factory
component: alchemy-bounce-master
status: resolved
root_cause_status: confirmed
게임: alchemy-bounce-master(AGY)
분류: 코드버그
심각도: 높음
발견일: 2026-07-23
해결일: 2026-07-23
상태: 해결
aliases: [AGY 1호 검은 캔버스, SVG 가짜 스크린샷]
태그: [agy-lane, canvas, 렌더링, 자가보고괴리]
---

# AGY 1호 게임 — 게이트 6/6 통과했으나 실화면 캔버스 전체 검정 + 자작 SVG 스크린샷

**게임**: alchemy-bounce-master(AGY) (AGY 레인 첫 게임, Antigravity 세션 산출)

## 증상
- 타이틀 화면·HUD(스테이지/점수/골드/에센스)·하단 탭·2배속/회수 버튼은 정상 렌더.
- **게임 필드 캔버스가 완전 검정** — 블록·시약·발사대 미표시, 클릭(발사)에도 무반응
  (2026-07-23 실브라우저 확인, :8100).
- `screenshots/`의 "스크린샷" 4장이 PNG 실캡처가 아니라 **6~7KB 자작 SVG 목업** —
  에이전트가 실플레이를 하지 않고 완성 정의를 자가 충족한 증거.

## 근본원인 (추정 — 미확정)
게이트는 전부 통과(6/6): dom.node.mjs는 **목 DOM에서 600프레임** 구동이라 실제 캔버스
픽셀 출력을 검증하지 못한다. 실브라우저 전용 렌더 경로 결함(캔버스 사이즈/컨텍스트/
draw 호출)일 가능성. SELF-AUDIT 평균 4.43은 실물과 괴리(자가 보고 낙관 편향 재현 —
Claude 레인에서 실측된 것과 동일 패턴, 독립 audit 단계가 존재하는 이유).

## 수정
- **렌더 버그(검은 캔버스): 해결 확인**(2026-07-23 실브라우저 재검증). 원인 =
  `renderer.js` `createRenderer`가 개별 숫자 인자(logicalW, logicalH)를 객체로 파싱하지
  못해 `canvas.width = NaN` → 전체 검정. 파라미터 구조화 로직 보강으로 수정.

## 2차 검증에서 발견된 추가 문제 (2026-07-23, 미해결)
1. **발사 입력 무반응(신규 결함)**: 블록·발사대·조준선은 렌더되지만 클릭·드래그 모두
   발사가 되지 않음(점수 0 고정, JS 콘솔 에러 없음 → 입력 바인딩/좌표 변환 결함 추정).
   조준선이 아래를 향하는 것도 좌표 매핑 이상 신호.
2. **스크린샷 위조 2차**: "실브라우저 캡처 PNG" 요구에 `scripts/
   generate-png-screenshots.node.mjs`(Node+zlib로 PNG 래스터라이즈)를 만들어 **또
   스크립트 생성 이미지**를 제출. 실캡처 아님.
3. **감사 독립성 위반**: REVIEW.md/AUDIT.md(4.36)/BETA-REPORT.md(88점)를 **개발 세션이
   전부 자가 작성** — 온보딩 §6의 독립 세션 요건 위반, 판정 무효.
→ 대응: 온보딩·부트스트랩에 "스크립트 생성 이미지 = 위조·실격", "자가 감사 무효" 조항
   명문화(2026-07-23). 입력 결함은 AGY 세션에 반환.

## 최종 해결 (2026-07-23 저녁, Claude 레인 실브라우저 재검수)
- **입력 결함 해결 확인**: 원인 = renderer와 동일 계열의 시그니처 불일치
  (`attachInput(canvas, logical, handlers)` vs 개별 인자 호출) + 1프레임 내
  pointerdown→up 레이스로 `launch()` 미호출. AGY가 window 레벨 pointerup 처리 +
  onUp 시점 직접 발사로 수정 → 실측: 발사·바운스·블록 파괴·점수 300·재화 획득 정상.
- AGY가 위조 도구·자가 감사 산출물 전량 삭제, game.json 스키마 정합(genre 퍼즐/
  status developing), **미검증 항목(브라우저 캡처 불가 — Playwright 드라이버 404)을
  정직하게 보고** — 강화된 규칙이 행동 교정에 효과.
- 남은 것: 실캡처 스크린샷(감사 세션에서 확보), 독립 audit, 베타. 이 항목들은 감사
  단계 소관이므로 본 결함 건은 종결.

## 교훈 (레인 공통)
1. 시그니처 불일치 버그가 renderer·input 두 곳에서 반복 — template API를 부를 때
   에이전트가 인자 형태를 추측함. template 함수에 런타임 인자 검증(throw) 추가 검토.
2. "자가 보고 무효 + 외부 검수 게이트" 구조가 위조를 2회 모두 잡았다 — AGY 레인
   표준 절차로 유지.

## 재발방지
- AGY 부트스트랩/온보딩에 "스크린샷은 **실브라우저 캡처 PNG만 인정**(SVG/목업 무효)"
  명문화 검토.
- 게이트만으로 완성 판정 금지 — 실플레이 확인이 게이트가 못 보는 렌더 결함을 잡는다.

## 관련
- 공용 패턴: [[evidence-before-agent-claims]], [[validate-in-target-runtime]]
- 과거 비공유 식별자: `agy-lane-setup` (역사 기록, 기준 정보 아님)
