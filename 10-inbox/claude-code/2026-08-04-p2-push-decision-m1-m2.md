---
type: session-note
schema_version: 1
project: embolos
status: active
created: 2026-08-04
agents: [claude-code]
review_by: 2026-10-04
tags: [app-track, push-notifications, eas, expo, build-system-decision]
---

> 2026-09-04 인박스 검토: M3(앱 측 expo-notifications·등록 훅) 미완이라 승격 근거 부족 →
> `review_by` 2026-08-11 → 2026-10-04 재지정. embolos 저장소에서 M3 상태 확인 후 승격/보관 결정.

# P2 푸시 알림 — 구조 결정 승인 + M1·M2 구현 (진행 중)

## 사용자 결정 (2026-08-04 승인 — 권장안 전부)

1. **Expo Go → EAS development build 전환** — 근거: SDK 53+부터 Expo Go는 원격 푸시
   미지원(공식 FAQ가 정본, changelog의 "iOS 유지" 서술은 발표 시점 잔재). **운영 변화**:
   앱 실측 절차가 "Expo Go 스캔"에서 "dev build 설치"로 바뀐다.
2. **Android 먼저**(FCM V1 서비스 계정 = Firebase 무료), iOS(Apple Developer $99/년) 보류.
3. **Expo Push Service 경유**(exp.host) — 백엔드는 httpx 직접 호출(Python SDK는 PyPI
   구버전 방치·패키지 혼선으로 배제). 신규 시크릿 불필요(토큰이 곧 주소).
4. 1차 트리거 3종: order_paid(구매자+셀러)·shipping_started(구매자).

명세: embolos `docs/app/p2-push-plan.md` (4-에이전트 조사 근거 포함).

## 진행 (백엔드 legal 브랜치)

- M1 `e386a75`: 0075 device_push_tokens(RLS 밖·양축 XOR·소유 이전 유니크) +
  push_notifier(100건 청크·DeviceNotRegistered 즉시 무효화) + /api/push/register·unregister
  (aud 판정 후 기존 검증기 위임 — 게이트 복제 금지). 테스트 6.
- M2 `e2d8548`: message_dispatch push 채널(recipient=소유 축 — flush가 발송 시점 토큰
  재해석) + 트리거 3종 배선. 푸시는 자동알림 토글 없이 기본 on(기기 등록=수신 의사표시),
  토큰 0건이면 로그 무생성. 테스트 5.
- 남은 것: M3(앱 — expo-notifications·등록 훅·딥링크, EAS 자격증명 전 dormant-safe),
  M4(영수증 cron·잔여 트리거·수신 토글). **eas init·Firebase 프로젝트는 사용자 액션.**

## 운영 함정 (이번에 확인)

- Expo web은 백엔드 CORS 부재로 앱 검증 경로가 아님 — 네이티브만. 재시도 금지.
- 이 세션 환경에서 장기 백그라운드 프로세스(에뮬레이터·uvicorn)가 3회 일괄 외부 중단 —
  실측은 사용자 참관 세션 또는 수동 실행으로.
