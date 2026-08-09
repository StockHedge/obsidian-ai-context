---
type: project-card
schema_version: 2
project_id: embolos
project: Embolos
status: active
repo_path: C:\Users\jihon\projects\embolos
branch: legal-kakaopay-2026-07
head_commit: aaaa76c
updated: 2026-08-09
last_verified: 2026-08-04
agents: [cursor, codex, claude-code]
tags: [embolos, multi-tenant-saas, ops, billing]
---

# Embolos — 프로젝트 카드

## 한 줄 목적

판매자별 상점, 구매 흐름, 운영 기능을 제공하는 멀티테넌트 AI 쇼핑몰 SaaS 플랫폼이다.

## 기준 위치

- 로컬 저장소: `C:\Users\jihon\projects\embolos`
- 현재 브랜치: `legal-kakaopay-2026-07`
- 확인한 HEAD: `aaaa76c` (앱 저장소 embolos-app HEAD `67471e1`)
- 원격 추적: 현재 브랜치가 `origin/legal-kakaopay-2026-07`을 추적
- 프로젝트 `AGENTS.md`: 2026-07-27 Cursor v2 마이그레이션으로 신설
- 현재 상태 `docs/ai/NOW.md`: 2026-07-27 신설 (실측 Git·헬스 기준)
- Cursor 어댑터: `.cursor/rules/shared-context.mdc`
- 기존 상세 인계: `docs/현재_상태_핸드오프.md`, `docs/handoff/pc-laptop-sync.md`

프로젝트 저장소의 코드·Git·최신 검증 결과가 이 카드보다 우선한다. 기존 인계 문서는 일부 시점 정보가 오래됐으므로 단독 기준으로 사용하지 않는다.

## 구조 지도

- `backend/`: FastAPI, Jinja2, SQLAlchemy, Alembic 기반 API와 서버 렌더링
- `frontend/`: React·Vite 기반 관리 화면
- `design/`: 제품·UI 설계 자산
- `docs/`: 앱, 운영, 결제, 광고, 오케스트레이션 및 인계 문서
- `.github/workflows/`: 상태 점검과 예약 작업

## 핵심 제약

- 하나의 DB에서 테넌트를 분리하며 애플리케이션의 tenant scope와 RLS를 함께 지킨다.
- 앱 연결은 pooled Neon async 경로, 마이그레이션은 direct DB 경로를 사용한다.
- 마이그레이션 실행 책임은 API에 있고 Ops 앱은 별도 마이그레이션을 실행하지 않는다.
- 상업 `/admin`과 별도 Ops 콘솔의 UI·배포·쿠키 경계를 유지한다.
- Ops 자동실행 화이트리스트는 비어 있으며 제안만 생성한다.
- 구독 직접 변경과 카카오 결제 트리거는 사람 승인 없이 실행하지 않는다.
- 사용자 문구는 프로젝트의 한국어 카피 규칙을 따른다.

## 현재 단계

### 확인된 현재 상태 (2026-08-04, 노트북→PC 인계 시점)

- 완결 누적: W4 예약(0072) · T1~T3 혜택 · 앱 P0~P3+푸시 P2(자격증명·dev build 포함) ·
  AI 분석 트랙 P1~P3(0070~0078) · 커스텀 필드 0065(0079) · 쿠폰존 큐레이션 D5(0080)·D13.
- 배포: **prod v37 = 0078 라이브**(2026-08-04, release 로그 실증) · 코드 HEAD = 0080
  (다음 배포에 0079·0080 동반). 카카오페이 CID 활성(구독 실청구). prod Ops는 여전히 미배포.
- CI: tests+parity green(최종 push분). 양 저장소 클린·완전 동기.

### 남은 트랙 (정본은 repo `docs/ai/NOW.md` «다음 행동»)

- 사용자 게이트 4건: 실기기 푸시 검증 · main 병합(analytics cron 발화 조건) ·
  다음 배포 0079·0080 동반 · serial/coupon kind 결정 2건(`docs/ai/plans/serial-coupon-kind-plan.md`)
- 백로그: 구 6탭 철거 · 셀러 고객 상세 화면 · 앱 체크아웃 커스텀 필드 파리티 ·
  레거시 order_form_fields 이관 · MessageSendLog 원문 보존 · Ops prod 승격 · 광고 대안(AdFit 사전문의)
- 2026-07-27 목록에서 이월(2026-08-04 갱신본에 누락): 가격·무료 티어의 실제 캡과 게이팅 ·
  만료 테넌트 purge 활성화 전 운영 조건 확인

## 중요 사건

- [[incidents/2026-07-31-branch-security-review-w4|브랜치 보안 리뷰 W4 — 확정 5건 전량 수정]]
- [[incidents/2026-08-01-pytest-prod-db-near-miss|pytest가 prod DB로 향한 니어미스 — PYTEST_DB_OK 가드 도입]]

## 주요 마일스톤

- [[2026-07-26-ai-company-ops-console|AI Company Ops Console 구현·test 검증]]
- [[milestones/2026-07-28-beta-acquisition-kit|핸드메이드·리빙 5인 베타 모집 기반]]
- [[milestones/2026-07-28-beta-tester-operations-playbook|베타테스터 실무 플레이북]]
- [[milestones/2026-08-03-benefit-track-t1-t3-complete|혜택 트랙 T1~T3 완결]]
- [[milestones/2026-08-03-app-p1-benefit-parity|앱 P1 혜택 파리티 완결]]
- [[milestones/2026-08-04-ai-analytics-track-p1-p3|AI 분석 트랙 P1~P3 완결]]
- [[milestones/2026-08-04-push-custom-fields-benefit-backlog|prod v37 + 푸시 후속 + 커스텀 필드 0065 + 혜택 백로그 D5·D13]]

## 관련 공통 패턴

- [[evidence-before-agent-claims]]
- [[symptom-is-not-root-cause]]

## 마지막 검증

- 확인일: 2026-08-04 (Claude Code — 노트북→PC 인계 세션)
- 확인한 근거: Git 실측(양 저장소 클린·원격 동기 0/0·HEAD 해시), GH Actions CI green(tests
  16m49s+parity), prod v37 release 로그(0075→0078 리비전 라인), 스모크(apex·health 200,
  신규 cron 403, /seller/ai 303), 테스트 DB alembic current=0080
- 확인하지 못한 항목: 실기기(푸시 수신·앱 신규 화면 — 에뮬레이터는 노트북 환경 결함으로 금지),
  콘솔 신규 화면 브라우저 육안(입력 항목·쿠폰존 보드 — 컴파일 게이트·라우트 테스트로만 고정),
  외부 콘솔 설정(Cloudflare·카카오페이·OAuth)
- 상시 기준: repo `docs/ai/NOW.md` · `PROGRESS.md`(머신 전환 절차) · `AGENTS.md`
- 이력: 임시 Vault 인계 문서는 repo `docs/ai/`·`docs/ai_company_ops.md`로 흡수돼 2026-07-27
  정리됐다. 2026-07-27 검증(콜드스타트로 `test.embolos.kr/health` 타임아웃 해소 등)의 상세는
  [[90-archive/2026-07-27-cursor-context-v2-apply]]에 남아 있다.
