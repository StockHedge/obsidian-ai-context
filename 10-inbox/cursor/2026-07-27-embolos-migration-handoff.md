---
type: session-handoff
schema_version: 1
status: pending-project-migration
project: embolos
agent: cursor
created: 2026-07-27
review_by: 2026-08-03
contains_verified_result: true
tags: [migration, handoff, uncommitted-work]
---

# Embolos 종합 마이그레이션 핸드오프

> **작성**: 2026-07-27 (KST) · Cursor 세션에서 작성  
> **목적**: 다른 PC/다른 AI 세션에서 **바로 이어서** 작업할 수 있게 상태·결정·미완·주의사항을 한곳에 모음  
> **프로젝트 루트**: `C:\Users\jihon\projects\embolos`  
> **브랜치**: `legal-kakaopay-2026-07` (origin 추적 중)  
> **옵시디언 vault**: `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

---

## 0. 인수 AI — 시작 전 필수 읽기 (5분)

1. 이 문서 전체를 읽는다.
2. repo에서 `git status` / `git log -5 --oneline` 로 **미커밋 규모**를 확인한다. (아래 §3 참고 — **대량 uncommitted**)
3. `docs/ai_company_ops.md` + `docs/오케스트레이션_논박_및_재구조안.md` 로 Ops 경계를 재확인한다.
4. 사용자에게 **다음에 무엇을 할지** 한 줄로 확인한 뒤 진행한다. (커밋/푸시/prod 배포는 명시 요청 없으면 하지 않음)
5. **마이그레이션이 완료되면 이 문서(및 §11의 포인터 노트)를 삭제한다** — 말미 §12 명시문 준수.

---

## 1. 한 줄 현황

Things #1–3·#5–7 제품면과 **#4 AI Company Ops 콘솔(별도 Fly 앱)** 구현·test 배포까지 진행됨.  
상업 `/admin`에 AI Company를 이식하지 않음. **코드 변경 상당수가 아직 git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>"/push 되지 않음.**

---

## 2. 고정 결정 (깨면 안 되는 것)

| 항목 | 결정 |
|------|------|
| #4 형태 | 플랫폼 페이지 이식 **금지** → 별도 Ops 콘솔 |
| 데이터 | Neon **0061 제어면 테이블 공유** (스키마 포크·이벤트버스 신설 금지) |
| 배포 | Fly `embolos-ops` / test `embolos-ops-test` |
| 호스트 | `ops.embolos.kr` · test `ops.test.embolos.kr` |
| 상업 `/admin` | Fleet·WorkOrders·Money·impersonation 유지 |
| Ops 인증 | 쿠키 `embolos_ops` (admin과 분리, `is_admin`/`admin_role` 재검증) |
| 자동실행 | 화이트리스트 **공집합** — proposal만. 승인 후 실행은 상업 `/admin` 딥링크 또는 Claude Code 세션 |
| 마이그레이션 소유 | **api 앱만** alembic. ops는 release_command에 마이그레이션 없음 |
| 런타임 AI | Ops Haiku 1 + CS 1(기존). 빌드면 PM/Dev/QA는 세션만(상주 워커 없음) |
| 금지 | CF Queues/D1, 7워커 상주, Ops의 구독 UPDATE·카카오 결제 트리거 |

계획 파일(`ai_company_ops_console_*.plan.md`)은 **편집하지 말 것** (사용자 지시).

---

## 3. Git / 작업 트리 상태 (매우 중요)

- 브랜치: `legal-kakaopay-2026-07` ↔ `origin/legal-kakaopay-2026-07`
- **Ops·제어면·Things 제품면 관련 변경이 working tree에 대량 존재** (modified + untracked).
- 이 핸드오프 작성 시점 기준으로 **해당 트랙 전체를 커밋했다는 보장이 없음**. 인수 직후 `git status`로 실측할 것.
- 대표 untracked/신규 영역(예시):
  - `backend/app/ops/`
  - `backend/Dockerfile.ops`, `backend/fly.ops.toml`, `backend/fly.ops.test.toml` (존재 여부 확인)
  - `backend/alembic/versions/0061_control_plane.py`, `0062_ai_global_pause.py`
  - `backend/app/admin/control_routes.py`, `flags_service.py`, `roles.py`, …
  - `.github/workflows/health-ping.yml`
  - Things 셀러 면: `seller_services.py` 등
- 사용자 **명시 요청 없이** commit / push / `--force` 하지 말 것.

---

## 4. 최근 완료된 작업 묶음

### 4-1. Things 제품면 (#4 제외, 대체로 완료로 보고됨)
- 셀러 허브 `/seller/services` 및 #1 디자인 / #2 이전 / #3 CS 에스컬레이션 / #5 앱 / #6 업셀 CTA / #7 AI·소비 지표
- 문서: `docs/things_product_surface.md` (repo에 있으면 따름)
- test (`embolos-test`)에 해당 표면 배포된 이력 있음

### 4-2. #4 AI Company Ops (S1~S3 계획 범위)
- 헌법·경계 문서 정착 (`docs/오케스트레이션_논박_및_재구조안.md`, `docs/ai_company_ops.md`)
- Ops FastAPI: 로그인·Mission·Alerts·Jobs·Proposals·SSE/폴링·진단
- W1 health-ping(GH) · W2 dead-man · W3 5xx 미들웨어 · W4 money-watch · W6 quota→업셀 proposal만
- S2 Ops digest(Haiku) + `ai_global_pause` (0062 시드)
- GH `scheduled-jobs.yml`: run-charges **월1**, domains-gc **일1**, purge-expired **보류**, ops 감시 잡 포함
- test 라이브: **https://ops.test.embolos.kr** (cert 발급됨), api `embolos-test`에 내부 `/internal/ops/*` 배포
- 모의장애: dead-man 발화 · W3 5xx 임계 히트 · 로컬 pytest 계약 테스트
- 플랜 to-do: docs-constitution / ops-scaffold / s1-watchers / s1-heartbeat / s2-ops-agent / s3-w6-billing-report → completed

### 4-3. 옵시디언 정리 (2026-07-27)
- vault 재구성: `00-home` … `90-archive`, 프로젝트는 `20-projects/embolos/`
- Cursor 인박스: `10-inbox/cursor/`
- 상세 세션 로그(마일스톤): `20-projects/embolos/milestones/2026-07-26-ai-company-ops-console.md`

---

## 5. 인프라·URL 스냅샷

| 대상 | 상태 | 비고 |
|------|------|------|
| `embolos-test` (api) | deployed | `OPS_BASE_URL=https://ops.test.embolos.kr` 반영 |
| `embolos-ops-test` | deployed | 시크릿을 test api에서 복제한 이력 |
| `ops.test.embolos.kr` | DNS+TLS OK | CNAME → `embolos-ops-test.fly.dev` (와일드카드보다 구체) |
| `embolos-ops` (prod 앱) | **생성만 / pending** | 시크릿·`ops.embolos.kr` DNS·배포 **미완** |
| `embolos-api` (prod) | 기존 | Ops 변경 미배포일 수 있음 — 확인 필요 |
| Neon/Redis | test·prod 분리 원칙 | ops는 **같은 환경의 api와 DB 공유** |

스모크(참고):
- `GET https://ops.test.embolos.kr/health` → ops ok
- `GET https://test.embolos.kr/health` 또는 `embolos-test.fly.dev/health` → api ok
- 온박스: `python /app/scripts/ops_deadman_smoke.py`, `ops_w3_smoke.py` (TEST_ENV 전용 5xx 시뮬 포함)

---

## 6. 코드·문서 지도 (어디를 보면 되는지)

### Repo 문서
- Ops 경계·체크리스트: `docs/ai_company_ops.md`
- 헌법: `docs/오케스트레이션_논박_및_재구조안.md`
- W1: `docs/ops_w1_health_ping.md`
- 상업 제어면: `docs/admin_control_plane.md`
- Things 제품면: `docs/things_product_surface.md` (있으면)

### 구현 위치 (이름만)
- Ops 앱: `backend/app/ops/` (main, routes, session, alerts, watchers, agent, error_rate_middleware, internal_routes, templates)
- api 마운트: `backend/app/main.py` (ErrorRateMiddleware + ops internal router)
- Fly: `backend/fly.ops.toml`, `fly.ops.test.toml`, `Dockerfile.ops`; api `fly.toml` / `fly.test.toml`에 `OPS_BASE_URL`
- CI: `.github/workflows/scheduled-jobs.yml`, `health-ping.yml`
- 플래그 시드: alembic `0062` (`ai_global_pause`)
- 제어면 스키마: alembic `0061` + `backend/app/models/control_plane.py`

### Obsidian
- 이 핸드오프: `10-inbox/cursor/2026-07-27-embolos-migration-handoff.md`
- 프로젝트 카드(구 리마인드): `20-projects/embolos/PROJECT-CARD.md` — **내용이 7/24 기준이라 Ops 이후 상태로 갱신 필요할 수 있음**
- Ops 마일스톤 로그: `20-projects/embolos/milestones/2026-07-26-ai-company-ops-console.md`
- 7/27 세션(로깅 정정): `10-inbox/cursor/2026-07-27_세션로그_컨텍스트로깅-경로정정.md`

---

## 7. 과금·플랜 경로 (Ops 턴마다 보고 의무)

| 경로 | 성격 |
|------|------|
| W6 → 업셀 `proposals` | **제안만**. 실행은 `/seller/billing` 또는 상업 `plan_upsell` + 사람 |
| Ops → `seller_subscriptions` UPDATE / 카카오 결제 | **금지** |
| prod `min_machines_running≥1` | 인프라 월 ~$3–6 — **사용자 승인 후** |
| Ops Haiku | 소액 + `ai_global_pause` |
| GH Actions / W1 | 대개 $0 |
| Claude Code | 빌드면 예산 (런타임과 분리) |

---

## 8. 다음에 하면 좋은 일 (우선순위 제안)

### P0 — 이관 직후 안전
1. `git status`로 미커밋 목록 실측 · 사용자에게 커밋/PR 여부 확인
2. test Ops 로그인·Mission/Alerts가 살아 있는지 브라우저 확인
3. GH: `APP_BASE_URL` / `CRON_SECRET` / `BILLING_CRON_SECRET` 가 환경에 맞게 켜졌는지 확인

### P1 — prod Ops 올려야 할 때 (사용자 승인 후)
1. `embolos-ops`에 prod 시크릿 복제 (api와 동일 Neon/Redis/JWT 등 — 값 로그 금지)
2. Cloudflare: `ops.embolos.kr` CNAME + ownership/ACME
3. `flyctl deploy -c fly.ops.toml -a embolos-ops`
4. prod api에 `OPS_BASE_URL=https://ops.embolos.kr` 배포

### P2 — 제품/백로그 (Ops 밖·병행)
- `PROJECT-CARD.md`에 남아 있던 축: 앱 P4(푸시) · 트랙 B 무료티어 · 광고 대안 · purge 활성화 · 카카오페이 CID/legal · OAuth 콘솔
- Ops 후속: W5·W7, 트랙션 게이트 후 자동실행 **재검토만** (배포·머니·시크릿 영구 HITL)

### 하지 말 것
- 상업 `/admin`을 7워커 대시보드로 비대화
- Ops에 셀러 WorkOrder UX·impersonation·구독 CRUD 이식
- 시크릿을 채팅/커밋/로그에 평문 노출

---

## 9. 환경·도구 메모 (Windows)

- Shell: PowerShell. `curl`는 `curl.exe` 권장 (alias 충돌).
- 한글 파일 I/O: UTF-8 `Path.write_text` / 임시 .py 패턴 (`ps-korean-safe-write` 스킬).
- Fly SSH 종료 시 Windows에서 "handle is invalid"가 나와도, 본문 출력이 정상이면 스모크 성공으로 본 적 있음.
- 사용자 Cursor 전역: 한국어 우선, 검증 루프, 시크릿 커밋 금지, 파괴적 git 금지.

---

## 10. 관련 Cursor plan

- 이름: AI Company Ops Console  
- 로컬 plan 경로 예: `C:\Users\jihon\.cursor\plans\ai_company_ops_console_61370b92.plan.md`  
- **플랜 파일 편집 금지.**

---

## 11. 포인터

인박스 포인터 노트(같은 마이그레이션 묶음):  
`10-inbox/cursor/2026-07-27_마이그레이션-핸드오프-포인터.md`  
→ 본 문서로 안내만 함. **마이그레이션 완료 시 본 문서와 함께 삭제.**

---

## 12. 마이그레이션 완료 시 삭제 지시 (필수 · 인수 AI 준수)

**이 문서의 수명**

- 본 노트(`10-inbox/cursor/2026-07-27-embolos-migration-handoff.md`)는 **일시적 이관용**이다.
- 영구 지식은 repo `docs/*`, `PROJECT-CARD.md`, milestones/세션 로그로 흡수한 뒤에만 정리한다.

**인수(이어받는) AI가 직접 해야 할 삭제 조건**

다음을 **모두** 만족하면, 사용자에게 한 줄로 “마이그레이션 핸드오프 문서를 삭제한다”고 알린 뒤 **인수 AI가 직접 삭제**한다.

1. 인수 AI가 이 문서의 내용을 바탕으로 작업 컨텍스트를 복원했고,
2. 사용자(또는 인수 세션)가 이관이 끝났다고 인정했거나, 동등한 상시 문서(PROJECT-CARD / repo docs)로 상태가 반영되었으며,
3. 더 이상 이 임시 핸드오프에 의존하지 않는다.

**삭제 대상**

- `10-inbox/cursor/2026-07-27-embolos-migration-handoff.md` (**이 파일**)
- `10-inbox/cursor/2026-07-27_마이그레이션-핸드오프-포인터.md` (포인터)

**삭제하지 말 것**: milestones·세션 로그·PROJECT-CARD·repo docs (영구 기록).

> **명시**: 마이그레이션이 완료되면, **전달받는 AI가 위 삭제 대상을 직접 삭제**한다.  
> 사용자에게 “삭제할까요?”만 묻고 미루지 말고, 완료 조건이 충족되면 **삭제 실행까지** 수행한다. (사용자가 삭제를 명시적으로 금지한 경우에만 예외.)
