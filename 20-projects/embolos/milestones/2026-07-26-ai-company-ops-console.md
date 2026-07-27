---
type: milestone
schema_version: 1
project: embolos
status: completed
created: 2026-07-27
work_date: 2026-07-26
agents: [cursor]
verification: test-environment
tags: [ops-console, fastapi, fly, monitoring, billing-guardrails]
---

# Embolos — AI Company Ops Console 마일스톤

- **작성**: 2026-07-27 (세션 작업 본편: 2026-07-26)
- **도구**: Cursor AI
- **프로젝트**: Embolos (`C:\Users\jihon\projects\embolos`)
- **계획(편집 금지)**: Cursor plan `ai_company_ops_console_61370b92`
- **용도**: 구현 범위, 검증 결과, 남은 위험을 보존한 마일스톤 기록

---

## 1. 사용자가 요청한 것

### 1-1. Things #4(AI Company) 재정의
- 플랫폼 `/admin`이나 셀러 콘솔에 “AI Company” UI를 **이식·확장하지 말 것**.
- Downloads의 오케스트레이션 논박/재구조안(헌법)을 **기준 문서**로 삼을 것.
- 데이터/스키마(0061 제어면 테이블)는 공유하되, **UI·배포 단위는 별도**로 둘 것.
- 7워커 상주, CF Queues/D1, 이벤트버스 신설, Chief of Staff 중계 LLM, 자동실행 화이트리스트 등은 **기각** 쪽으로 맞출 것.

### 1-2. 승인된 플랜 전량 구현
- 첨부된 *AI Company Ops Console* 플랜을 명세대로 구현할 것.
- **플랜 파일 자체는 수정하지 말 것**.
- 이미 만들어진 to-do를 다시 만들지 말고, 진행하면서 상태만 갱신할 것.
- 첫 항목부터 `in_progress`로 표시하고, **전부 완료할 때까지 멈추지 말 것**.
- 슬라이스: S0는 전제(재구현 금지), S1 하드닝 착수 → S2 Ops 에이전트 → S3 W6 업셀 proposal(실행 아님).

### 1-3. 과금·플랜 경로 보고 의무
- Ops/AI Company 구조화 턴이 끝날 때, 생길 수 있는 과금·플랜 변경 접점을 **사용자에게 반드시 보고**할 것.
- Ops 에이전트·감시자가 `seller_subscriptions`를 직접 UPDATE하거나 카카오 결제를 트리거하는 것은 **금지**.

### 1-4. Obsidian 컨텍스트 로깅 (후속 지시, 2026-07-27)
- 잘못된 vault 경로에 쓴 노트(`Embolos/AI-CONTEXT-LOGGING.md`)는 **지울 것**.
- 올바른 경로: `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
- “A–Z까지”는 알파벳을 억지로 채우라는 뜻이 **아님**.
  - 요청한 작업과 어떻게 진행됐는지를 **리스팅해서 디테일하게** 작성하라는 뜻.

---

## 2. 진행 경과 (상세)

### 2-1. 문서·경계 고정
- 헌법을 repo `docs/오케스트레이션_논박_및_재구조안.md`로 정착.
- 호스트·앱 경계·S1 체크리스트·과금 보고 의무를 `docs/ai_company_ops.md`에 작성·갱신.
- 상업 제어면(`/admin`, Fleet·WorkOrders·Money)과 Ops 콘솔의 역할을 문서상 분리.
- Obsidian Things 노트에 “#4 = 별도 ops 콘솔 + 헌법 링크” 요약을 追記.

고정 결정(플랜 기본값 그대로 적용):
- 배포: 별도 Fly 앱 `embolos-ops` / test `embolos-ops-test`
- 호스트: `ops.embolos.kr` · `ops.test.embolos.kr`
- DB/Redis: 기존 Neon·Upstash 공유 (스키마 포크 금지)
- 인증: `embolos_ops` 쿠키 (admin 세션과 분리, 동일 admin 권한 재검증)
- 자동실행: 화이트리스트 공집합 — proposal만

### 2-2. Ops 앱 스캐폴드 (S1)
- backend 쪽에 Ops 전용 FastAPI 엔트리 구성: 로그인, Mission, Alerts, Jobs, Proposals, 이벤트(SSE/폴링), 진단(run-now, 가역·감사 성격).
- Ops용 Dockerfile + `fly.ops.toml` / `fly.ops.test.toml`.
- 마이그레이션 소유권은 api만 — ops release_command에 alembic 없음.
- 상업 `/admin`에 AI Company 대시보드를 붙이지 않음.

### 2-3. 감시자·경보 (S1~S2 배선)
- W3: api 미들웨어에서 5xx를 Redis 구간 카운터로 모아 `platform_alerts` 승격 (P1/P2, fingerprint 약 30분 억제). 조회는 ops UI.
- W2: `job_runs` 기대 주기 카탈로그 기준 dead-man → 최근 성공 부재 시 P2.
- Telegram: `OPS_BASE_URL` 기준 ops 딥링크 포함. test/prod fly env에 `OPS_BASE_URL` 반영.
- W4: 청구 실패·past_due 등 머니 신호를 경보로 승격하는 내부 잡.
- W1: Fly 밖 GitHub Actions health-ping + 설정 문서 (`docs/ops_w1_health_ping.md`).
- 내부 트리거: api `/internal/ops/*` + `X-Cron-Secret` (dead-man / money-watch / quota-watch / digest).

### 2-4. 외부 cron 스케줄 정합 (S1)
- GitHub `scheduled-jobs.yml` 조정:
  - `billing/run-charges` → 월 1회
  - `domains/gc-pending` → 일 1회
  - `purge-expired` → 보류
  - ops dead-man/money는 빈번 슬롯, digest/quota는 일 1 슬롯
- W1용 `health-ping.yml` 유지·문서화.

### 2-5. S2 Ops 에이전트 + pause 플래그
- Haiku 기반 일 다이제스트 + P1/P2 시 가설을 `proposals`(source=`ops_engine`)로만 출력. 실행권 0.
- `ai_global_pause` 글로벌 플래그 시드(마이그레이션 0062, 기본 꺼짐). 켜면 digest 스킵.
- CS는 기존 support 에이전트 유지 — ops에 재구현하지 않음.

### 2-6. S3 W6 (게이트 전 준비 포함 구현)
- AI/티어 한도 근접·초과 → 상향 전환 proposal만.
- 셀러 구독 변경은 사람 승인 후 기존 `/seller/billing` 또는 상업 `/admin` `plan_upsell` 경로로만.
- 계약 검증용 테스트에 “구독 UPDATE·카카오 결제 없음”을 명시.

### 2-7. test 배포·인프라
- Fly 앱 `embolos-ops-test` 생성·배포.
- `embolos-test`(api)에 감시자·내부 ops·`OPS_BASE_URL`·0062 포함 재배포.
- test 시크릿을 ops-test로 복제 (값을 로그에 남기지 않음).
- Cloudflare DNS: `ops.test.embolos.kr` CNAME(와일드카드보다 구체), ownership TXT, ACME challenge.
- TLS 발급·활성 확인. `https://ops.test.embolos.kr/health` OK, 로그인 페이지 200.
- prod 앱 `embolos-ops`는 생성만 — 시크릿·DNS·배포는 미완.

### 2-8. 모의장애·검증
1. dead-man: 온박스 스모크 호출 → 기대 cron 성공 부재로 다수 P2 발화 확인.
2. W3: TEST_ENV 전용 5xx 시뮬로 임계 이상 히트 확인.
3. 로컬 pytest(계약/임계/딥링크) 통과.
4. Windows `fly ssh` 종료 시 handle is invalid는 결과와 무관한 클라이언트 노이즈로 처리.

### 2-9. 턴 종료 과금 보고 (사용자에게 전달한 내용)
- W6 → 업셀 proposal: 제안만. 구독 변경은 사람 + 기존 billing/plan_upsell
- Ops → 구독 UPDATE / 카카오 결제: 금지·미구현
- prod min_machines_running≥1: 권고만, 월 약 $3–6급 — 승인 후 fly.toml
- Ops Haiku: 소액 런타임 LLM + ai_global_pause
- GH Actions / W1: 대개 $0
- Claude Code: 빌드면 예산 — 런타임 오케스트레이션과 분리

### 2-10. 플랜 to-do 마감
- docs-constitution · ops-scaffold · s1-watchers · s1-heartbeat · s2-ops-agent · s3-w6-billing-report → 모두 completed.

---

## 3. 아직 남은 것 (이 세션에서 안 한/못 한 것)

- prod `embolos-ops` 시크릿 복제 · `ops.embolos.kr` DNS · 배포
- GitHub Variable `APP_BASE_URL` / Cron Secrets가 test·prod에 맞게 켜져 있는지 사용자 측 확인
- W5(외부 API 실패율)·W7(보안 시그널) 후속
- 트랙션 게이트(유료≥30 또는 월주문≥1k) 이후 자동실행 승격 재검토만 (배포·머니·시크릿은 영구 HITL)
- 이 트랙에 대한 git commit --trailer "Co-authored-by: Cursor <cursoragent@cursor.com>"/PR — 사용자 미요청이라 만들지 않음

---

## 4. 한 줄 요약

Things #4를 상업 `/admin` 이식이 아니라 공유 0061 DB + 별도 Fly Ops 콘솔으로 재정의·구현했고, S1 하드닝·S2 digest/pause·S3 업셀 proposal(실행 없음)까지 test에 올렸으며, 과금은 제안·HITL만 남기고 구독 직접 변경은 금지로 닫았다.

---

## 5. 빠른 참조 (경로만)

- 헌법: `docs/오케스트레이션_논박_및_재구조안.md`
- Ops 경계: `docs/ai_company_ops.md`
- W1: `docs/ops_w1_health_ping.md` · `.github/workflows/health-ping.yml`
- Cron: `.github/workflows/scheduled-jobs.yml`
- test Ops: https://ops.test.embolos.kr
