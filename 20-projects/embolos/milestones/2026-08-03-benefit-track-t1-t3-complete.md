---
type: milestone
schema_version: 1
project: embolos
status: completed
created: 2026-08-03
work_date: 2026-07-29 ~ 2026-08-03
agents: [claude-code]
verification: full-suite-888 + browser-verified
source_repo: StockHedge/embolos
source_commit: 413db19
tags: [benefit-engine, checkout, money-path, public-surface, coupon-zone, badges]
---

# Embolos — 혜택 트랙 T1·T2·T3 전체 완결

## 무엇이 끝났나

죽어 있던 0064 혜택 엔진(스키마·순수 계산기·테스트 52건은 있는데 런타임 소비 0)을
3단계로 완전 개통:

- **T1 빌더**: ORM↔엔진 어댑터(benefit_loader) + 체크아웃 배선(reserve/confirm/release
  원장) + 콘솔 빌더 3화면 + 진단 패널("미리보기 = 실주문과 같은 함수").
- **T2 이관**: 구세대 3세계(Coupon·DiscountOffer·loyalty rules)를 0074 백필로
  benefits에 흡수, 계산기를 하나로. 구세대 화면·라우트 은퇴, 쿠폰 만료 cron 재조준.
- **T3 공개면**: 체크아웃 혜택별 라인·place 정합 가드 3종·주문 후 4화면 산식
  partial·이메일 영수증 / 상품 뱃지(콘솔 저작→SSR 4+1면) / 쿠폰존 `/coupons`·진입
  동선·threshold 넛지·정책 zone 탭. 전체 스위트 888 상당 그린, 브라우저 실측 통과.

## 보존할 설계 자산 (재사용 가치)

- **표시 금액 = 청구 금액**: 노출 후보(뱃지·넛지·쿠폰존)도 청구 계산기와 **같은
  함수 호출**을 지난다. 표시 계층의 산수 사본 금지.
- **D7** 뱃지 = qty=1 가상 라인에 대한 엔진 실호출의 applied 승자(priority 첫 매칭은
  패자 뱃지 사고).
- **D8** 거절 사유 allowlist(fail-closed) — "무시할 거절"을 나열해야 미래 축이
  자동으로 안전 방향.
- **D9** 반사실 검증 — 그 사유의 문턱**만** 눕힌 재계산에서 생존해야 조건부
  표시/넛지("따라 담았는데 안 붙음" 구조 차단). 두 문턱 동시 완화는 틀린 약속.
- **D12** 라벨 이원 — 주문 전=라이브 이름, 주문 후=원장 스냅샷(소급 변경·CASCADE 방어).
- **D14** 코드 진열대에 금액 슬롯 금지.
- **fail-open의 구조적 차단**: 코드 혜택은 CHECKOUT_KINDS에서 제외하고
  code_benefit_ids로만 후보 진입 — "코드를 안 냈는데 적용"이 SQL 수준에서 불가능.

## 잔여 (백로그)

serial/coupon kind 개통(`_reapply_reversed` 한도 재검증 선행) · 앱 네이티브 뱃지
파리티(D13) · 쿠폰존 수동 큐레이션(D5) · gift 지급.
