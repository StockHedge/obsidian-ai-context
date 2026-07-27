---
type: project-card
schema_version: 2
project_id: embolos
project: Embolos
status: active
repo_path: C:\Users\jihon\projects\embolos
branch: legal-kakaopay-2026-07
head_commit: 459a5c0
updated: 2026-07-27
last_verified: 2026-07-27
agents: [cursor, codex]
tags: [embolos, multi-tenant-saas, ops, billing]
---

# Embolos — 프로젝트 카드

## 한 줄 목적

판매자별 상점, 구매 흐름, 운영 기능을 제공하는 멀티테넌트 AI 쇼핑몰 SaaS 플랫폼이다.

## 기준 위치

- 로컬 저장소: `C:\Users\jihon\projects\embolos`
- 현재 브랜치: `legal-kakaopay-2026-07`
- 확인한 HEAD: `459a5c0`
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

### 확인된 현재 상태

- 최신 확인 커밋 `459a5c0`은 별도 AI Company Ops 콘솔, 제어면, Things 제품면 작업을 포함한다.
- `docs/ai_company_ops.md`와 [[2026-07-26-ai-company-ops-console]]에 따르면 test Ops 배포와 S1~S3 범위가 구현됐다.
- 작업 트리에 `docs.zip`과 `docs/handoff/`가 추적되지 않은 상태로 남아 있다. 검토 전 임의 커밋·삭제하지 않는다.

### 문서상 남은 트랙

- 앱 P4: 푸시 알림과 앱스토어 마감
- 가격·무료 티어의 실제 캡과 게이팅
- 광고 대안 검토
- 만료 테넌트 purge 활성화 전 운영 조건 확인
- 카카오페이 CID와 OAuth 콜백 등 외부 콘솔 의존 작업
- Ops prod 배포 상태, W5·W7 후속의 최신 재검증

이 목록은 여러 시점의 문서에서 모은 후보이며 다음 작업 시작 때 코드·Git·외부 환경으로 다시 확인한다.

## 중요 사건

- 현재 Vault에 Embolos 전용 사건 노트 없음

## 주요 마일스톤

- [[2026-07-26-ai-company-ops-console|AI Company Ops Console 구현·test 검증]]

## 관련 공통 패턴

- [[evidence-before-agent-claims]]
- [[symptom-is-not-root-cause]]

## 마지막 검증

- 확인일: 2026-07-27
- 확인한 근거: 저장소 Git 상태·브랜치·최근 커밋, `README.md`, `docs/ai_company_ops.md`, 앱 계획, 오케스트레이션 문서
- 확인한 작업 트리: `docs.zip`, `docs/handoff/` 미추적
- 확인하지 못한 항목: 운영 서비스의 현재 배포 상태, 외부 콘솔 설정, 실제 결제·OAuth 상태
- 임시 Vault 인계: [[2026-07-27-embolos-migration-handoff]]
- 다음 구조 작업: 프로젝트 저장소에 `AGENTS.md`와 `docs/ai/NOW.md`를 도입한 뒤 임시 Inbox 인계를 정리
