---
type: milestone
schema_version: 1
project: game-factory
component: matgo-pocket
status: monitoring
verified: 2026-08-11
agents: [claude-code]
source_repo: various-game/matgo-pocket
tags: [play-console, iarc, gambling-simulation, beta-gate, first-pass]
---

# matgo-pocket — 파이프라인 최초 완주작, 스토어 제출 직전 도달

## 무엇을 달성했나

포켓 맞고(`kr.stockhedge.matgopocket`)가 공장 파이프라인 전 단계를 **최초로**
통과하고 Play 스토어 제출 2단계 전까지 도달했다 (2026-08-11).

- 킥오프 90분 타임아웃 구출 → dev → 게이트 8종 → 감사 4.14 → **베타 71.4 합격**
  (베타 게이트 통과는 공장 역사상 처음 — 이전 시도는 전부 인프라 단절이나 점수 미달)
- 콘솔 등록(4974659319720121897) · 등재 4언어 · AAB production 업로드(vc2)
- 필수 설문 완료: 데이터 보안, 광고 ID, **IARC 콘텐츠 등급**(08-11)
- registry `consoleSetup.surveysDone=true` 기록 (registry API 경유 — game.json
  직접 수정은 위조 지문 검증에 걸린다)

남은 2단계는 국가/지역 선택 + 게시 개요 "변경사항 전송" (사람/브라우저 몫).

## 검증 근거

- 베타 71.4: `lastBeta.runId=20260810-1831-local-1` (game.json)
- IARC 등급 산출 스크린샷 확인: PEGI 18(도박 이미지) / USK 16(사행성 테마) /
  IARC Generic 18+ — 콘솔 요약 페이지에서 육안 검증 (2026-08-11)
- 앱 콘텐츠 "주의 필요" 탭: "모두 확인했습니다" (잔여 선언 0)

## 사행성 카테고리의 특수 경로 (재사용 교훈)

도박 질문에 예=핵심 선언 시:

1. **대한민국 자동 배포 불가** — IARC 요약에 "대한민국에서 제공되지 않습니다.
   GRAC에 이메일(rc.globalratings@grac.or.kr)로 문의" 표시. 한국 GRAC은 사행성
   모사 게임을 IARC 자동 등급 위임에서 제외하며, 한국 출시는 GRAC 직접
   등급분류가 별도로 필요하다. 맞고 게임인데 한국 배포가 막히는 아이러니.
2. 허위 선언(도박 없음)으로 우회하면 사후 적발 시 계정 제재 위험 — 정직 선언이
   유일 경로. 설문 자동화 시트가 무해 템플릿 답안을 복사한 것을 입력 전 검증으로
   잡아 정정한 사례가 이번 세션에 있었다(자동 루프가 도박 질문을 "아니요"로
   오염 → 전수 덤프 검증으로 발견·정정).

## 브라우저 자동화 재확인 사항

- IARC 설문의 MDC 체크박스는 상태가 `mdc-checkbox--selected` 클래스에 있다 —
  `aria-checked` 셀렉터는 라디오(`material-radio`)에만 유효. 같은 페이지라도
  컴포넌트별 상태 표현이 달라 검증 셀렉터 재사용이 오탐을 만든다.
- 긴 JS 루프는 렌더러 프리즈(45초 타임아웃)를 유발 — 그룹 단위 분할 실행 필요.

## 관련 커밋과 문서

- 저장소 상태: `various-game/game-factory/docs/ai/NOW.md` (2026-08-11 갱신)
- 아이콘 전면 거부 사건: incidents `2026-07-29-play-rejection-default-launcher-icon`
- 설문 답안 출처: `matgo-pocket/store-assets/console-survey.md` (사행성 정정 3건)
