---
type: project-card
schema_version: 2
project_id: youth-investment
status: active
repo_path: C:\youth-investment
remote_url: https://github.com/StockHedge/YouthInvestment
branch: main
head_commit: 11bc802a (origin/main, 2026-09-02)
updated: 2026-09-02
last_verified: 2026-09-02
agents: [claude-code]
tags: [fintech, education, simulation-trading, fastapi, expo]
---

# FinPle (youth-investment)

## 한 줄 목적

청소년 모의투자 플랫폼 **FinPle** (운영명 핀쉐어/FinShare). 실거래 없이 자체 체결 엔진으로
모의투자 + 금융 교육(레벨/퀴즈/미션/랭킹)을 제공한다.

## 기준 위치

- 로컬 저장소: `C:\youth-investment` (main)
- 원격 저장소: https://github.com/StockHedge/YouthInvestment
- 세션 인계 문서: 저장소 `docs/PROJECT_STATUS.md` (이 repo는 `docs/ai/NOW.md` 대신 이 관례를 사용)
- 운영 백엔드: `https://finple-backend.fly.dev` (Fly.io nrt) · 운영 DB: Neon Singapore

## 개발자 테스트 계정 (2026-08-18 신설)

- **email `devtester@finple.dev` / password `FinPle!Dev2026`** — 운영·로컬 공통 QA 계정.
- 성격: SMS 인증 우회 + 거래시간 가드 면제. 가상 자산만 보유하는 저위험 테스트 계정.
- 구현: `backend/app/core/dev_account.py` — `DEV_ACCOUNT_EMAIL/PASSWORD` env(fly secrets)가
  설정된 환경에서만 startup upsert. 비밀번호는 secrets 값 변경 + 재배포로 회전.
- 비활성화: `fly secrets unset DEV_ACCOUNT_EMAIL DEV_ACCOUNT_PASSWORD` 후 재배포
  (계정 row 는 남으나 가드 면제·비번 동기화가 꺼진다).
- 참고: 평문 비밀번호 기록은 사용자 명시 지시(2026-08-18)에 따른 것 — 시크릿 기록 금지
  정책의 예외이며, 서비스 공개 확대 전 회전 권장.

## 구조 지도

- `backend/` — FastAPI async + SQLAlchemy 2.0 + Alembic(마이그레이션 022, 다음 023) + Neon PG + Redis(fallback). API 라우터 18, 서비스 26, pytest 129.
- `YouthInvestmentExpo/` — 현행 앱. Expo SDK 55 / RN 0.83 / React 19.2 / TS 5.9. 화면 12 도메인.
- `YouthInvestment/`, `mobile/` — RN 0.73 레거시 잔재. 삭제 후보.
- `.github/workflows/` — backend-ci / frontend-ci / backend-deploy(수동 트리거).

## 핵심 제약 (변경 금지로 합의된 결정)

- ETF 미지원 — 청소년 위험 노출 우려로 완전 제거 (PR #33).
- 시세 소스는 **KIS(한국투자증권) Open API** — REST 현재가 + WebSocket 실시간, 모의(paper) 도메인. 키움은 PR #65에서 완전 제거.
- 브랜드: 슬로건 "투자, 배움부터." / Toss Blue `#0064FF` 단일 accent.
  **마스코트는 2026-08-17 '코니'(동전+새싹)로 전면 통일** (앱 아이콘 + 인앱 전부, 로고 제외).
  Figma 시안 5종 중 E안 사용자 채택 (파일: figma.com/design/C3WoK9x61GWpfsHdngfPP3).
  구현 교훈: 마스코트가 단일 컴포넌트(Mascot/Pinpi.tsx)로 중앙화돼 있어 내부 벡터 교체만으로
  11개 화면 + 일러스트 5종이 무수정 통일됨 — 중앙화된 브랜드 자산의 가치 실증.
- 매수=빨강, 매도=파랑 (한국 증시 컨벤션).

## 현재 단계

**출시 전 안정화 — 리뷰 Medium 마감 · 웹 재구조 3단계 · EOD 하드닝 · W/A2 (2026-09-02, PR #105~#124)**

- 리뷰 Medium 4건(refresh 회전+denylist·웹 CSP·N+1 배치·시즌 자동 정산), 웹
  데스크톱 재구조 1~3단계(랜딩·Auth 스플릿·G마켓 산스·루트 `DESIGN.md`·앱 셸
  컨테이너), EOD 첫 실전 결함 3건 + 하드닝(벌크 업서트 19분→13초·`eod_window`·
  백업 크론), W 스트릭 3축, A2 운영자 콘솔(사용자 검색·푸시 이력, 마이그 030).
- 게이트: pytest **291/291** · tsc 0 · alembic 운영 리비전 **030** (다음 031).
- 인계 기준 문서: 저장소 **`docs/ai/NOW.md`** (이력은 `docs/PROJECT_STATUS.md` 아크 표).
- 결정 대기: `price_history` 중복 207건(000320 단일 종목) 정리 + UNIQUE 마이그 031.
- 관찰 대기: 백업 크론 첫 발화, 운영자 콘솔 A2 실확인(ADMIN_TOKEN 필요),
  카카오 실계정 왕복(사용자 수동).
- 8/18 E2E 잔여 이슈 4건(교육 본문 빈 값, 홈 지수 위젯 0, edge-to-edge 탭바,
  퀴즈 미답 안내)은 여전히 미수정 기록 상태.

이전 (2026-09-01, PR #91~#104): 카카오 로그인 복구, 자금·미션 무결성(마이그 027),
보안 3건(마이그 028), NAV 스냅샷 파이프라인(마이그 029), 종목상세 다크모드.

## 중요 사건

- 2026-08-17 발견: 로컬 working tree가 2026-03-24에 정체(원격 대비 94커밋 뒤처짐, PR #32~#68 미반영).
  로컬 유일 커밋은 venv site-packages 통째 커밋(11,705파일/220만 줄)으로 보존 가치 없음.
  → 같은 날 RegisterScreen 패치 백업 후 origin/main 으로 정렬 완료 (사용자 승인).
- 2026-08-17 발견: CI 자동 배포가 빌드 컨텍스트 오류("/requirements.txt not found")로
  한 번도 성공한 적 없던 상태 — `working-directory: backend` 픽스로 해소, run 성공 검증.
- 2026-08-18 발견(운영 E2E): ① access 토큰 15분 만료 후 refresh 미사용으로 인증 API
  전면 401 — 자동 갱신 인터셉터로 수정, 운영 로그 실증. ② 종목상세 차트의 웹 SVG
  소문자 `<text>` → 네이티브 Render Error 로 화면 진입 불가 — 웹에선 우연히 동작해
  은폐. 교훈: **환경 분기(웹/네이티브, DEBUG 분기)의 프로덕션 쪽 가지는 실환경 E2E
  없이는 커버리지 사각지대** (SMS 500 결함과 동일 패턴 재확인).

- 2026-08-30~09-01: 카카오 로그인이 출시 차단 수준으로 3중 사망(빌드 미주입 /
  시크릿 미전송 / URI 미등록) + portfolio_snapshots 0행으로 기능 2개 무음 사망을
  발견·복구. 교훈: **"기능이 있다"는 코드 존재가 아니라 데이터·번들 실측으로
  판정** (번들 dead-code, 0행 테이블 모두 코드 리뷰로는 안 보였음).
  상세: [[10-inbox/claude-code/2026-09-01-finple-kakao-review-ledger-nav-darkmode]]

## 주요 마일스톤

- **2026-09-01: 코드리뷰 대장 마감 + NAV 파이프라인** — PR #91~#104, 인박스 세션노트 참조.
- **2026-08-17: 출시 게이트 완료** — [[milestones/2026-08-17-launch-gate]] 참조.
- 2026-06-03: KIS Phase 3~5 완료 (WS 실시간 시세 + 키움 제거), PR #61~#68 머지.
- 2026-06-01: 백엔드 운영 배포 완료 (Fly.io + Neon + release_command 마이그레이션).
- 2026-05-27: bold-direction UI 전면 리디자인(14화면) + CI/CD 스캐폴딩.

## 관련 공통 패턴

- (미기록 — venv 커밋 방지/.gitignore 선행 정비는 30-patterns 승격 후보)

## 마지막 검증

- 확인일: 2026-09-02
- 확인한 근거: pytest 291/291 실행, 운영 /health 200, Neon SQL 로 alembic 리비전
  030·9/2 price_history 2,729행·NAV source=EOD 1행 직접 조회, 웹 CSP 응답 헤더·
  콘솔 위반 0, 운영 API 연동 dev 브라우저 실화면(랜딩·Auth 스플릿·홈 스트릭 위젯·
  차트 컨테이너), 마이그 030 Neon 임시 브랜치 왕복 검증.
- 확인하지 못한 항목: 백업 크론 발화(9/2 22:00 KST 이후), 운영자 콘솔 A2 실화면
  (ADMIN_TOKEN), 카카오 실계정 왕복, 모바일·태블릿 실기기, EAS 네이티브 빌드.
