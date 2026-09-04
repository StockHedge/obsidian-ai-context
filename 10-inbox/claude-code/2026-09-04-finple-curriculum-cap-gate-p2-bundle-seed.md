---
type: session-note
schema_version: 1
project: youth-investment
status: monitoring
root_cause_status: confirmed
created: 2026-09-04
agents: [claude-code]
review_by: 2026-09-11
tags: [finple, education, curriculum, investment-cap, stamp-bootstrap-seed-loss, psycopg2-executemany, fly-proxy-compression, redundant-indexes, metro-bundle-split, platform-stub, neon-branch-verification, eod-provisional-data]
---

# FinPle — 커리큘럼 결정 구현 · 한도 게이트 · P2 성능 · 번들 분할 · 시드 통합 (PR #125~#148)

## 무엇을 했나 (검증: pytest 318/318 · Neon SQL · 운영 API/브라우저 실화면 · Pages/Fly 헤더 실측, 2026-09-02~04)

세션 3회 아크(09-02 밤 ~ 09-04 오후). 저장소 `docs/PROJECT_STATUS.md` 2026-09-01~04 절에
PR 별 표, `docs/ai/NOW.md` 인계, 사용자 결정은 `docs/DECISIONS_PENDING_2026-09-04.md`.

- **교육**: 안전자산 5강 Level 5~9(#127) → Level 3·4 stub 교체(#129) → Level 10 주문 이해(#138)
  → **투자 한도 게이트**(#139·#140, L0~2 강제 해제 → 한도 100만/300만/무제한) → 행동 연동
  안내형(#146) → 시드 단일 카탈로그화(#143). 교육 컨텐츠 0~20 이 코드 원천·매 배포 upsert.
- **운영 DB**: `price_history` 중복 207행 정리 + UNIQUE(마이그 031, #126), 중복 인덱스 40개
  DROP(마이그 032, #131 — 32→20 MB). Neon 임시 브랜치에서 upgrade→downgrade→upgrade 검증.
- **성능**: API gzip, Pages 해시 자산 immutable, 웹 초기 번들 1,775→1,411 KB(#134).
- **EOD**: UPDATE 경로 execute_batch + 갱신 필드 분포 로그(#125). 9/2·9/3 발화 패턴 실측.

## 사건·교훈 (승격 후보)

### 1. 데이터 마이그레이션이 운영에 닿지 않았다 — 두 번째 반복 (confirmed)

- 증거: Neon SQL `education_contents` 21행 중 **Level 0~4 만 존재**(2026-09-02). 마이그 021
  (2026-05, PR #45)의 Level 11~20 INSERT 는 운영 빈 DB 부트스트랩(`create_all` + `alembic
  stamp head`)에서 실행된 적이 없음. 8/20 미션 시드 유실(007/020)과 **같은 근본원인**인데 그때
  INSERT 를 가진 마이그를 전수 점검하지 않아 021 이 빠졌다.
- 처리: 시드성 데이터는 `scripts/init_db._seed_essentials` 의 카탈로그 upsert 로 이관
  (`app/seeds/education_catalog.py`, #127·#143). 릴리스 로그로 `inserted/updated/unchanged`
  가 남는다. alembic 은 스키마 변경 전용(CLAUDE.md 체크리스트에 명문화).
- 재사용 교훈(30-patterns 후보): **stamp 부트스트랩 환경에서는 "마이그 안 INSERT" 를 전수
  grep 해 시드 경로로 옮기고, 한 번 잡힌 유실은 같은 클래스 전체를 점검한다.** 기존
  [[30-patterns/ai-reliability/env-sourcing-optin-guard]] 류와 별개 항목.

### 2. 결정 문서의 "현행" 서술이 코드와 달랐다 (confirmed)

- 8/22 결정 문서는 "교육↔거래 게이트가 하나도 없다" 고 썼지만, `enforce_prerequisites` 가
  L0~2 미완료 시 주문을 거부하고 프런트도 그 코드를 게이트 UI 로 처리하고 있었다(강한
  게이트). 결정(라: 한도 연동)을 그대로 얹으면 이중 게이트가 되므로 사용자 재확인 →
  L0~2 강제 해제·한도 대체.
- 교훈: **결정지 작성 시점의 "현행" 은 착수 시점에 코드로 다시 확인**한다. 특히 "없다" 류
  부정 진술은 grep 한 줄로 검증 가능하다. [[30-patterns/ai-reliability/point-in-time-state-is-not-steady-state]] 와 같은 계열(다른 세션 초안).

### 3. SQLAlchemy "벌크" 는 문장 유형별로 다르다 (confirmed)

- 9/2 21:10 KST 스케줄 실행이 갱신 1,300행에 3분 36초. psycopg2 `executemany_mode` 기본
  `values_only` 는 INSERT 만 `execute_values` 로 접고 UPDATE 는 행당 왕복(1,300 × ~165ms
  = 실측과 일치). `values_plus_batch` 로 전환(#125). 직전 세션의 "19분→13초" 측정은 INSERT
  만 있던 날의 값이었다.
- 교훈(30-patterns 후보): 원격 DB 배치는 **문장 유형별 왕복 수**로 검증한다.

### 4. "Fly 프록시는 압축하지 않는다" 는 틀린 전제였다 (confirmed, 정정)

- `Accept-Encoding: br` 요청에 `content-encoding: br` 이 돌아옴 — 앱은 brotli 를 만들지
  않으므로 프록시 압축. 다만 낮은 레벨: 72 KB 목록이 프록시 br 15.3 KB vs 앱 gzip 9.1 KB.
  앱 gzip 유지가 맞고, PR 본문·주석의 전제를 정정했다. 교훈: **압축 유무는 identity/gzip/br
  세 요청의 wire 크기로 판정**(content-type 만 보고 단정 금지).

### 5. Metro 의 `try { require('expo-x') }` 는 웹 번들 제외 수단이 아니다 (confirmed)

- 소스맵 실측: expo-notifications+expo-device+의존 133 KB 가 웹 번들에 포함. `.web.ts`
  플랫폼 스텁으로 모듈 그래프 자체를 제외(#134). 교훈: 플랫폼 분기는 **파일 확장자**로.
  워크트리+junction 에서 `expo export` 하면 폰트 경로가 `assets/___<proj>/node_modules` 로
  escape 돼 wrangler 가 제외 — 정식 웹 배포는 main 에서.

### 6. EOD 잠정치 창 (monitoring)

- 9/2: 20:17 KST dispatch 적재분 대비 21:10 실행에서 47% 갱신, 21:41 안정. 9/3: 16:30 본
  실행 미발화, 21:10 첫 실행 2,728행 신규, 이후 갱신 0. KRX 당일 데이터는 ~21시 이전에
  확정되는 것으로 보이며, 16:30 본 실행이 제때 발화하는 날의 `갱신 필드 분포` 로그가 남은 질문.

## 검증된 결과

- 운영: alembic 032, `price_history` 315,339행·중복 0, 인덱스 20 MB, `education_contents`
  21행(0~20), `investment_cap` 응답·`unlocks`/`action` 라벨, 웹 번들 br 390 KB, 청크 immutable.
- 로컬 E2E(SQLite+DEBUG+Redis 컨테이너, 교육 0건 계정): 200만 거절 → 50만 체결 → 60만
  누적 거절, 웹 게이트 UI·CTA. 운영 E2E(dev 계정): L5 퀴즈 통과 → CTA → 예·적금 화면.

## 미검증·추정

- 16:30 본 실행이 제때 발화하는 날의 필드 분포(close/volume) — 미관측.
- 운영자 콘솔 A2 실화면(ADMIN_TOKEN), 모바일 뷰포트·실기기, 카카오 실계정 — 사용자 몫.

## 다음 작업

- `docs/DECISIONS_PENDING_2026-09-04.md` 의 A-1~A-5 답 → A-2 → A-5 → A-1 순 착수.
- Vault 인박스 2건 처리(사용자 결정), 이 노트의 승격 후보 검토.

## 승격 후보

- 사건: `20-projects/youth-investment/incidents/2026-09-02-education-seed-not-in-prod.md`(1번)
- 마일스톤: `20-projects/youth-investment/milestones/2026-09-04-curriculum-cap-gate.md`(커리큘럼 결정 구현 3 PR + 시드 통합)
- 공통 패턴: `30-patterns/deployment/stamp-bootstrap-drops-data-migrations.md`(1번), `30-patterns/performance/sqlalchemy-bulk-is-per-statement-type.md`(3번), `30-patterns/ai-reliability/verify-decision-doc-current-state-against-code.md`(2번)
