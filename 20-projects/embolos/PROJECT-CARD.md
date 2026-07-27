---
type: project-card
schema_version: 1
project_id: embolos
project: Embolos
status: active
repo_path: C:\Users\jihon\projects\embolos
updated: 2026-07-27
tags: [embolos, platform, ops, billing]
---

# Embolos — 프로젝트 카드

작성: 2026-07-24 (PC 이관 직후)
근거: `docs/현재_상태_핸드오프.md`(갱신 2026-07-13) + 이후 커밋(`legal-kakaopay-2026-07`) 대조
용도: 프로젝트의 큰 트랙을 찾기 위한 카드. 실제 현재 상태는 저장소의 `docs/ai/NOW.md` 또는 최신 handoff 문서를 우선한다.

---

## 이미 끝난 것 (문서보다 커밋이 앞섬)

- **스튜디오 다중 페이지 P4 1~5단계** — `store_pages` · 템플릿/AI 온램프 · SEO · 페이지 수 플랜 게이팅
- **통합 앱 P0~P3** — 구매자 JWT · 마켓 · 구매 · 셀러 모드 API
- 구 핸드오프의 에디터 Phase A/B/C · 리치텍스트 · prod 홀드 해제

---

## 남아 있는 Phase / 트랙

### 1. 앱 P4 — 푸시·마감
- 문서: `docs/app/embolos-app-plan.md`
- Expo Push 전면(주문 상태·신규 주문·마케팅 옵트인)
- 앱스토어 심사 준비 · 스토어 등록
- ※ 웹 스튜디오 P4(다중 페이지)와 **별 트랙**

### 2. 트랙 B — 가격 / 무료 티어 실기능
- 결정 완료, **구현 미착수**
- 캡 · "made by embolos" 브랜드마크 · 하단 광고 바(체크아웃 미노출) · 게이팅
- 무료 캡 정확 수치 설계 포함

### 3. 광고 트랙 (리서치 후 경로 축소)
- 셀러 서브도메인 AdFit **수익 셰어 불가**(카카오 서면 확정 2026-07-20)
- 남는 경로: **플랫폼 페이지 우선 게재**(스토어 쪽은 dormant 정책)
- 미결: 네이버 애드포스트 · AFP(invite-only, 장기)
- 문서: `docs/research/ads-revenue-share-2026-07.md`

### 4. 트랙 D Phase 3 — 만료 테넌트 purge 활성화
- 코드 완료, `tenant_purge_enabled=False` **이중 안전 off**
- 백업 = 이메일 첨부(RESEND) + 텔레그램 알림 (SA/Drive 폐기)
- 켜기 전: RESEND / ADMIN_ALLOWED_EMAILS / TELEGRAM_* / BILLING_CRON_SECRET 확인

### 5. 외부·대기 (사용자/콘솔)
- **카카오페이 CID 발급**(심사·서류) — 브랜치 `legal-kakaopay-2026-07`와 직결 · 활성 CID면 구독 실청구 주의
- 카카오/네이버 **test OAuth 콜백** 콘솔 등록 (`https://test.embolos.kr/oauth/*/callback`)
- **구글 소셜로그인**(소셜 Phase 2) — 라우트 미구현

### 6. 최신 WIP 잔여
- 커밋 `8c0b2e4`: 상품 쿠폰 스코프 테스트 + 정리 + 광고 리서치 — 마무리·정리 중일 수 있음

---

## 한 줄 요약

웹 스튜디오 P4는 사실상 완료. 남은 큰 축 = **앱 P4 · 트랙 B(무료 티어) · 광고 대안 · 카카오페이 CID/legal · purge 활성화 · 외부 OAuth/CID 대기**.

---

## 운영 메모 (이관)

- 작업 PC: `C:\Users\jihon\projects\embolos`, 브랜치 `legal-kakaopay-2026-07`
- 왕복: 자리 뜰 때 push / 이어받을 때 pull — `docs/handoff/pc-laptop-sync.md`
- 프로젝트 현재 상태와 설계 결정은 저장소 문서를 기준으로 유지한다.
- 이 Vault에는 검증된 사건, 마일스톤, 회고만 승격한다.

## Vault 기록

- [[2026-07-26-ai-company-ops-console|AI Company Ops Console 마일스톤]]

## 다음 정리

- 프로젝트 저장소에 `AGENTS.md`와 `docs/ai/` 구조 적용
- 적용 전 임시 인계: [[2026-07-27-embolos-migration-handoff]]
- 이 카드의 WIP 상태를 실제 Git과 최신 handoff 문서로 재검증
- 로컬과 원격의 동기화는 [[GIT-POLICY]]에 맞춰 운영
