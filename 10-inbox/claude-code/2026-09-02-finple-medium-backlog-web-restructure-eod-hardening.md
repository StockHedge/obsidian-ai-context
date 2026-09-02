---
type: session-note
schema_version: 1
project: youth-investment
status: monitoring
root_cause_status: confirmed
created: 2026-09-02
agents: [claude-code]
review_by: 2026-09-09
tags: [finple, refresh-rotation, csp, eod-batch, github-actions-cron, asyncpg, n-plus-one, web-restructure, design-md, streak, admin-console, migration-verification, worktree-junction]
---

# FinPle — 리뷰 Medium 마감 · 웹 재구조 3단계 · EOD 첫 실전+하드닝 · W/A2 (PR #105~#124)

## 무엇을 했나 (검증: 운영 실측·pytest 291/291·브라우저 실화면·Neon SQL, 2026-09-02)

이틀 아크(09-01 오후 ~ 09-02 밤). 저장소 `docs/PROJECT_STATUS.md` 2026-09-01~02 절에
PR 별 표, `docs/ai/NOW.md` 에 인계. 요지:

- 2026-08-23 리뷰 Medium 4건 마감: refresh 회전+denylist, 웹 CSP, portfolio N+1,
  시즌 자동 정산(SEASON_MDD_KEEP 결정 → 구현 채택).
- Variant 시안 기반 웹 데스크톱 재구조 1~3단계 완료 — 시안 카피의 사실 오류 교정
  (가상 500만원≠1,000만 원, FinPle Inc.≠핀쉐어, 미구현 기능, 비밀번호 필드 부재),
  G마켓 산스 display 전용·해요체·루트 `DESIGN.md` 신설·앱 셸 720 컨테이너.
- 9/1 EOD 배치 첫 실전에서 결함 3건 → 9/2 관찰에서 추가 3건 발견·수정(아래 사건).
- W(스트릭 3축)·A2(운영자 콘솔: 사용자 검색·푸시 발송 이력, migration_030) 마감.

## 사건·교훈 (승격 후보)

### 1. GitHub schedule 지연 + 배치 N+1 + 대상일 창 불일치 (EOD 파이프라인) — confirmed

- 증거: `gh run list` 스케줄 이력 — 8/24~26 35~40분, 8/27~9/1 **5~12시간** 지연.
  9/1 은 21:38 KST 발화(직전 세션 "미발화" 기록은 오진 — 최신 run 만 보는 Monitor
  가 dispatch run 에 가려진 것). 9/2 는 20시까지 미발화.
- 종가 수집 스텝 19분의 정체: 일괄 listing 7초 + **종목별 SELECT 2,770 왕복**
  (러너 미국→Neon 싱가포르 편도 ~150ms). 전 종목 skip 인 날도 동일. 벌크
  SELECT 1회 + executemany 로 → 13초 (9/2 실측 2,729행/10초). #120
- 잠복 버그: `import_today_close` 는 UTC `date.today()`, `snapshot_daily_nav` 는
  KST 날짜. 자정(KST)을 넘겨 발화하면 snapshot 이 D+1 을 봐 NAV 게이트가 조용히
  스킵. `app/services/eod_window.eod_target_date`(마감 후 확정 창 06:30~24:00 UTC,
  장중 None) 로 통일. #101 이후 처음 지연되는 날 터질 예정이었다.
- 대응: 백업 크론 19:00·22:00 KST (잡 멱등 + 2분이라 no-op 비용 미미, KRX IP
  차단도 다른 러너에서 자연 재시도).
- 교훈(패턴 후보): **저활동 저장소의 GitHub schedule 은 시간 단위로 밀린다 — 멱등
  잡 + 복수 크론 + 시각 창 게이트로 설계하라.** 배치 DB 접근은 왕복 수로 생각하라.

### 2. asyncpg `ssl=true` 거부 — 오진 2회 (9/1 밤) — confirmed

- 에러 문구 `sslmode parameter must be one of: ...` 가 URL 에 sslmode 가 남은 것처럼
  읽혀 bash 치환 → Python 정규화(#113)로 고쳤는데 같은 에러. 진범은 **`ssl` 파라미터
  값 자체를 sslmode 어휘로 검증**하는 것 — `ssl=require` 로 해결(#114).
- 검증 절차 확립: 로컬 최소 venv(Actions pip 목록 재현)에서 가짜 Neon 호스트로
  연결 시도 → 인증 단계 도달 여부로 파싱 통과를 판정. Actions 왕복 20분+과금 절약.
- 교훈: 에러 문구의 키워드가 가리키는 위치를 의심하라 — 값 검증 실패가 이름 오류
  처럼 보인다. 라이브러리 소스(`connect_utils._parse_connect_dsn_and_args`)로 확인.

### 3. 워크트리 강제 제거가 junction 을 따라가 main node_modules 를 비움 — confirmed

- CLAUDE.md 함정 표에 이미 "junction 부작용" 이 있었지만 **예방 절차가 없었다**.
  전날은 `rmdir` 로 junction 을 먼저 끊어 무사, 9/2 는 건너뛰어 707 패키지 소실 →
  `npm install` 복구(16초). CLAUDE.md PR 패턴 6단계에 `cmd //c rmdir` 선행 명시.
- 교훈(패턴 후보): Windows junction 은 "링크" 라는 직관과 달리 `git worktree
  remove --force`·`rm -rf` 가 target 을 지운다. 링크 제거는 `rmdir` 만.

### 4. 운영 DB 데이터 품질 발견 — price_history 중복 207건 (결정 대기)

- 노루홀딩스(000320) 단일 종목, 2025-07-18~2026-08-21, 값 동일 — 백필 2회 잔재.
  UNIQUE 제약 부재가 원인. 정리(DELETE 최소 id 외) + 마이그 031 UNIQUE 제안,
  파괴적 조작이라 사용자 결정 대기. import 스크립트는 중복 내성 확보(#120).

## 검증된 결과

- pytest 291/291 (신규 42: 회전 10·시즌 6·URL 1·EOD 11·스트릭 9·A2 6), tsc 0.
- 운영: backend health 200, alembic 리비전 030, 9/2 price_history 2,729행·NAV
  source=EOD 1행(Neon SQL 직접 조회), 웹 CSP 헤더·콘솔 위반 0, 랜딩/Auth 스플릿/
  홈 스트릭 위젯/차트 컨테이너 수납 브라우저 실화면(운영 API 연동 dev).
- 마이그 030: Neon 임시 브랜치에서 upgrade→downgrade→upgrade 왕복.

## 미검증·추정

- 백업 크론 첫 발화(9/2 22:00 KST 이후) — 다음 세션 확인.
- 운영자 콘솔 A2 실화면 — ADMIN_TOKEN 이 fly secret 이라 세션 진입 불가, 사용자 몫.
- 모바일 375·태블릿 768 실기기, 카카오 실계정, 로그인→refresh 실왕복 — 사용자 몫.
- Neon 임시 브랜치 `verify-mig-030-push-send-logs` 잔존 — 삭제는 사용자 확인 필요.

## 다음 작업

- 결정: price_history 중복 정리 + UNIQUE(마이그 031). 백로그 L → P2.
- 인박스 검토 트리거 도달: claude-code 인박스 5건 (8/1·8/4 는 review_by 경과).

## 승격 후보

- 사건: `20-projects/youth-investment/incidents/2026-09-02-eod-cron-delay-n1-date-window.md`
  (사건 1 — 근본원인 확인·재발 방지 완료)
- 공통 패턴: `30-patterns/github-schedule-idempotent-multi-cron.md`(사건 1),
  `30-patterns/windows-junction-worktree-cleanup.md`(사건 3)
- 마일스톤: 웹 데스크톱 재구조 3단계 완료 + DESIGN.md 신설 (#110·#116·#118)
