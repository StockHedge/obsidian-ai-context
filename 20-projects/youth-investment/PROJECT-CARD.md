---
type: project-card
schema_version: 2
project_id: youth-investment
status: active
repo_path: C:\youth-investment
remote_url: https://github.com/StockHedge/YouthInvestment
branch: main
head_commit: 04417663 (origin/main, 2026-09-04)
updated: 2026-09-04
last_verified: 2026-09-04
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

**출시 전 안정화 — 커리큘럼 결정 구현 · 투자 한도 게이트 · P2 성능 · 번들 분할 · 시드 통합 (2026-09-04, PR #125~#149)**

- 교육: 안전자산 5강(5~9)·Level 3·4 stub 교체·Level 10 주문 이해로 0~20 연속. **L0~2 강제
  게이트 해제 → 투자 한도 연동**(미완 100만 / L1 300만 / L2 무제한, 보유 원가 합산).
  행동 연동은 안내형(레벨별 "바로 해보기" CTA). 교육 컨텐츠는 코드 원천·매 배포 upsert.
- 운영 DB: `price_history` 중복 207행 정리 + UNIQUE(마이그 031), 중복 인덱스 40개 제거
  (마이그 032, 32→20 MB). API gzip, Pages 해시 자산 immutable, 웹 초기 번들 −20%.
- 게이트: pytest **318/318** · tsc 0 · alembic 운영 리비전 **032** (다음 033).
- 인계 기준 문서: 저장소 **`docs/ai/NOW.md`**, 사용자 결정 정본 **`docs/DECISIONS_PENDING_2026-09-04.md`**
  (포인트 경제·거래시간 문서·WS/상시 머신·실현손익 net·실제 잠금 + 사용자 직접 검증 + 운영 절차).
- 결정 불필요한 코드 작업은 소진. 다음 세션은 결정부터.
- 관찰 대기: 16:30 EOD 본 실행이 제때 발화하는 날의 `갱신 필드 분포`, 운영자 콘솔 A2
  실화면(ADMIN_TOKEN), 모바일 뷰·카카오 실계정(사용자).

이전 (2026-09-02, PR #105~#124): 리뷰 Medium 마감, 웹 재구조 3단계, EOD 하드닝, W/A2.

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

- 2026-09-02 발견: 운영 `education_contents` 에 Level 0~4 만 존재 — 마이그 021(2026-05)의
  Level 11~20 INSERT 가 stamp 부트스트랩에서 한 번도 실행되지 않음. **8/20 미션 시드 유실과
  같은 근본원인의 두 번째 반복**(그때 INSERT 마이그 전수 점검 누락). 시드성 데이터를
  `_seed_essentials` 카탈로그 upsert 로 전면 이관(0~20). 상세:
  [[10-inbox/claude-code/2026-09-04-finple-curriculum-cap-gate-p2-bundle-seed]]
- 2026-09-03 발견: 8/22 결정 문서의 "게이트 없음" 서술이 코드와 불일치(L0~2 강제 게이트가
  존재) — 결정(라)의 의미가 바뀌어 사용자 재확인 후 강제 해제·한도 대체. 교훈: 결정지의
  "현행" 은 착수 시 코드로 재검증.

## 주요 마일스톤

- **2026-09-04: 커리큘럼 결정 구현 + 한도 게이트 + P2** — PR #125~#148, 인박스 세션노트 참조.
- **2026-09-01: 코드리뷰 대장 마감 + NAV 파이프라인** — PR #91~#104, 인박스 세션노트 참조.
- **2026-08-17: 출시 게이트 완료** — [[milestones/2026-08-17-launch-gate]] 참조.
- 2026-06-03: KIS Phase 3~5 완료 (WS 실시간 시세 + 키움 제거), PR #61~#68 머지.
- 2026-06-01: 백엔드 운영 배포 완료 (Fly.io + Neon + release_command 마이그레이션).
- 2026-05-27: bold-direction UI 전면 리디자인(14화면) + CI/CD 스캐폴딩.

## 관련 공통 패턴

- (미기록 — venv 커밋 방지/.gitignore 선행 정비는 30-patterns 승격 후보)

## 마지막 검증

- 확인일: 2026-09-04
- 확인한 근거: pytest 318/318, Neon SQL(alembic 032 · price_history 315,339행·중복 0 ·
  인덱스 20 MB · education_contents 21행), 운영 API(`investment_cap`·`unlocks`·`action`),
  운영 웹 실화면(교육 트리 라벨·Level 3/5 상세 CTA·L5 퀴즈 통과 → 예·적금 화면), 로컬
  SQLite E2E(교육 0건 계정: 한도 거절/체결/누적 거절·게이트 UI), Pages/Fly 응답 헤더 실측,
  마이그 031·032 Neon 임시 브랜치 왕복 검증.
- 확인하지 못한 항목: 운영자 콘솔 A2 실화면(ADMIN_TOKEN), 모바일 뷰포트·실기기, 카카오
  실계정, EAS 네이티브 빌드, 16:30 EOD 본 실행의 필드 분포.
