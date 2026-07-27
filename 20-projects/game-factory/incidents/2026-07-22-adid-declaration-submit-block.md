---
type: incident
schema_version: 1
project: game-factory
component: tangle-out
status: resolved
root_cause_status: confirmed
게임: tangle-out
분류: 배포·콘솔
심각도: 높음
발견일: 2026-07-22
해결일: 2026-07-22
상태: 해결
aliases: [AD_ID 미선언 제출 차단, 광고 ID 선언 하드블록]
태그: [play-console, admob, ad-id, 제출게이트]
---

# 광고 ID(AD_ID) 미선언 → production 제출 버튼 비활성

**게임**: tangle-out (block-clear · order-pop은 사전 선언해 미발생 → **게임마다 별도 선언 필요**)

## 증상
앱 콘텐츠 설문 · 스토어 설정 · 라이브러리 AAB · 국가 176/177을 다 채웠는데도
"검토를 위해 변경사항 N개 제출" 버튼이 **회색(비활성)**.
order-pop식 등재정보 오탐과 증상이 비슷해 혼동 위험.

## 근본원인 (증상 ≠ 원인)
"문제 보기 → 발견된 문제 1개 → **불완전한 광고 ID 선언**".
AdMob(Google Mobile Ads SDK)이 매니페스트에 `com.google.android.gms.permission.AD_ID`를
자동 병합 → "광고 ID 사용" 의미인데, **앱 콘텐츠 → 광고 ID 선언**이 미완료.
Android 13+ 타겟 아티팩트는 이 선언 없이는 출시 불가(정책 게이트).

## 수정 (콘솔 전용, API 미지원)
앱 콘텐츠 → 광고 ID → "선언 작성" → **"예"** → 용도 **"광고 또는 마케팅"만** 체크
(앱 기능 / 애널리틱스 / 개발자 커뮤니케이션 오선택 금지 = 데이터 안전 선언과 일관) → 저장.
→ 제출 버튼 활성화됨.

## 재발방지
- 신규 AdMob 게임 첫 production에서 **제출 버튼이 회색이면 등재정보보다 AD_ID 선언을 먼저 확인**.
- 렌더러 타임아웃 중 좌표 클릭 밀림 주의 — 체크박스는 클릭마다 스크린샷으로 상태 검증.

## 관련
- 공용 패턴: [[symptom-is-not-root-cause]]
- 과거 비공유 식별자: `play-prereview-adid-declaration-block`, `new-app-first-production-flow` (역사 기록, 기준 정보 아님)
