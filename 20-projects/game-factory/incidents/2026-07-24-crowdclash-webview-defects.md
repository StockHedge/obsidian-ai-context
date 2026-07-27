---
type: incident
schema_version: 1
project: game-factory
component: crowd-clash
status: active
root_cause_status: suspected
게임: crowd-clash(AGY)
분류: 코드버그
심각도: 치명
발견일: 2026-07-23
해결일:
상태: 진행중
aliases: [크라우드 클래시 WebView 진행불가, 검증환경 불일치]
태그: [agy-lane, webview, capacitor, 베타, 검증환경]
---

# crowd-clash: WebView 전용 결함 3종 → 베타 2회 연속 불합격 → escalated

**경과**: 풀오토 3사이클(dev→review→audit→autoship→베타). audit는 3.86→4.14로 상승
합격했으나 **베타 26.5/100 → 라운드 소진 → escalated(beta_failure)**.

## 결함 (베타 실기기 실측, 2라운드 동일 재현)
1. **[치명] 메인 화면 전체 입력 무반응(진행 불가)** — 첫 화면 검은 배경 + 탭/스와이프 무응답
2. **[치명] Game Over의 Retry 탭 시 외부 Chrome으로 이탈** — Capacitor 내비게이션 설정 문제
3. HUD/레벨 표시 `{n}` 미치환 — 1호와 동일한 i18n params 버그 클래스 3번째 재발

## 근본원인 (구조)
**검증 환경 불일치**: dev·review·audit·자가검증이 전부 데스크톱 브라우저(Playwright)에서
수행되어 전부 통과 — 결함은 **Android WebView에서만** 발생. 통합 가설: WebView에서 JS가
부팅 초기에 크래시 → i18n 치환·입력 바인딩·화면 전환이 연쇄로 죽음(증상 1·3을 동시 설명).
APK 빌드 신선도는 검증 완료(무관) — dev의 "수정"이 데스크톱에서만 유효했던 것.

## 다음 액션 (escalated 인계)
1. 에뮬레이터에 최신 debug APK 설치 → `adb logcat | grep -iE "chromium|console|error"` 로
   WebView JS 크래시 특정 (chrome://inspect 병행 가능)
2. 표적 수정 dev 1발 → 대시보드 베타 버튼으로 수동 재시험 (라운드 카운터는 리셋됨)

## 재발방지 (파이프라인 제안 — 양 레인 공통)
- **에뮬레이터 스모크 게이트**: autoship 후·베타 전, "APK 설치→부팅→타이틀 탭 1회 반응"
  자동 확인. 데스크톱-WebView 갭을 베타(60분·라운드 소모) 전에 잡는다.
- i18n params 게이트(t() 호출의 params 완전성 정적 검사) — 3회 재발로 승격 필요.
