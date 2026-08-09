---
type: milestone
schema_version: 1
project: embolos
status: completed
created: 2026-08-04
work_date: 2026-08-03
agents: [claude-code]
verification: parity-17 + anchors-90 + ci-green + local-api-smoke + tsc-clean
source_repo: StockHedge/embolos + StockHedge/embolos-app
source_commit: d12d623 (backend) / ceeefdf (app)
tags: [app-track, benefit-parity, json-api, display-equals-billing]
---

# Embolos — 앱 P1 혜택 파리티 완결

## 무엇이 끝났나

혜택 T1~T3가 SSR에만 있고 JSON API는 혜택 무지 상태이던 갭을 4건으로 폐합:

1. **상품 뱃지** — `/api/market/stores/{sub}`·`products/{sub}/{slug}`에 `benefit_badge`
   (SSR과 같은 로더+판정 재사용, 카드=품절 사다리 서버 미러 / PDP=sale_status 게이트).
2. **카트 사전 견적** — `GET /api/shop/{sub}/cart/quote?code=`: `_totals_with_benefits`
   6튜플 + 표시 헬퍼 3종의 JSON화. 산수 사본 0.
3. **쿠폰존 JSON** — `zone_entries` 재사용, D14(금액 슬롯 금지)는 스키마에 필드 부재로 강제.
4. **주문 원장 라인** — `redemption_rows`(D12 스냅샷) + 조정행 + loyalty_spent_cents,
   coupon_code 병기는 code_id 게이트를 서버에서 적용.

앱(embolos-app): PDP·카드 뱃지(6색 매핑), 쿠폰존(체크아웃 `?code=` 프리필), QuoteSummary
(quote 응답만 사용), 주문 영수증(SSR `_order_money_rows.html` 동형 규칙).

## 세션 중 발견·봉합한 실결함 (재사용 교훈)

- **A3② 가드가 SSR place에만 있고 앱 checkout API에 없었다** — 엔진 거절 코드로 정가 주문
  침묵 생성 + coupon_code 오병기. `checkout.code_engine_error` 단일 진입점으로 승격(양쪽 공유).
  교훈: 같은 불변식을 지키는 가드는 표면(SSR/API)마다 있는지 **표면 전수로 대조**해야 한다.
- **전액환불 주문의 할인 라인 증발**(리뷰 에이전트 HIGH) — release_benefits가 원장을
  reversed로 전이 → API benefit_rows 빈 배열, SSR엔 `elif discount_cents` 방어 폴백이 있었음.
  서버 합성 행으로 동형 봉합. 교훈: SSR 템플릿의 **방어 분기(elif 폴백)도 API 파리티 대상**이다.

## 검증

신규 파리티 테스트 17 + 앵커(특성화·wiring·code·가드)·인접 90 passed + push 후 CI
(tests·parity) green + 로컬 uvicorn+시드 스토어(p1-demo) 4 API 실응답 + 앱 tsc 클린.
잔여: 앱 네이티브 실측(에뮬레이터)·뱃지 hex 실기기 대비 — 세션 환경에서 장기 백그라운드
프로세스 반복 중단으로 미완(수동 실측 절차는 저장소 PROGRESS.md에 기록).
