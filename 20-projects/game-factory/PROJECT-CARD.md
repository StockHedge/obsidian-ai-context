---
type: project-card
schema_version: 1
project_id: game-factory
project: AI 게임공장
status: active
repo_path:
updated: 2026-07-27
aliases: [AI 게임공장 버그로그, 게임공장 결함 대시보드]
tags: [game-factory, android, deployment, ai-pipeline]
---

# AI 게임공장 — 프로젝트 카드

`various-game` 게임 팩토리(블록 클리어 · 순서 팝 · 탱글 아웃 + game-factory 파이프라인)를
개발·배포하며 발생한 **모든 에러 / 버그 / 오진 / 수정 코드**를 한곳에 축적한다.
목적은 단순 기록이 아니라 **빈도 패턴을 읽어 재발을 줄이고 진단→수정 효율을 높이는 것**.

## 기준 위치

- 로컬 저장소: 미확인 (`various-game` / `game-factory` 관련 저장소)
- 현재 작업: 저장소의 `docs/ai/NOW.md`를 도입한 뒤 그 문서를 기준으로 사용
- 중요 사건: `incidents/`
- 공통 작성 정책: [[WRITING-POLICY]]

## 기록 규약

- 새 사건 1건은 `incidents/` 아래 노트 1개로 기록한다.
- 새 노트는 [[INCIDENT.template]]을 사용한다.
- 본문은 **증상 → 증거 → 실패한 접근 → 근본원인 → 수정 → 검증 → 재발방지** 순서를 사용한다.
- 추정 원인과 확인된 원인을 구분한다.
- 코드 전체를 복사하지 않고 파일 경로와 커밋을 우선한다.

## 현재 사건 (2026-07-27 검토)
| 날짜 | 게임 | 분류 | 심각도 | 제목 | 상태 |
|---|---|---|---|---|---|
| 07-21 | 공통(광고) | 코드버그 | 높음 | [[2026-07-21-rewarded-reward-type-empty\|보상형 reward.type='' → 컨티뉴 미작동]] | 해결 |
| 07-22 | tangle-out | 배포·콘솔 | 높음 | [[2026-07-22-adid-declaration-submit-block\|광고 ID(AD_ID) 미선언 → 제출 버튼 비활성]] | 해결 |
| 07-22 | order-pop | 배포·콘솔 | 보통 | [[2026-07-22-listing-prereview-false-positive\|사전검사 등재정보 기능묘사 오탐 차단]] | 해결 |
| 07-22 | game-factory | 프로세스오진 | 높음 | [[2026-07-22-edits-commit-403-permission\|edits.commit 403 = 서비스계정 권한 누락]] | 해결 |
| 환경 | 공통 | 환경설정 | 높음 | [[env-emulator-korean-path-panic\|에뮬레이터 한글 SDK 경로 부팅 패닉]] | 해결 |
| 환경 | 공통 | 환경설정 | 보통 | [[env-local-port-collision\|로컬 포트 공유 → 캐시가 게임 혼선]] | 해결 |
| 07-23 | alchemy-bounce-master(AGY) | 코드버그 | 높음 | [[2026-07-23-agy-black-canvas-fake-screenshots\|AGY 1호: 캔버스 검정+입력 무반응+스크린샷 위조 2회 (전부 교정)]] | 해결 |
| 07-23 | alchemy-bounce-master(AGY) | 코드버그 | 보통 | [[2026-07-23-agy-stageclear-placeholder\|스테이지 클리어 {gold}/{essence} 미치환→+0 표시 (payload 누락, 2단계 수정)]] | 해결 |
| 07-23 | game-factory | 배포·콘솔 | 높음 | [[2026-07-23-agy-headless-hang-zombies\|AGY 헤드리스 행: bare agy 좀비+고아 서버 (하트비트 자동정리로 해결)]] | 해결 |
| 07-24 | crowd-clash(AGY) | 코드버그 | 치명 | [[2026-07-24-crowdclash-webview-defects\|WebView 전용 결함 3종→베타 소진 escalated (검증환경 불일치)]] | 진행중 |

## 초기 인사이트
- **최다 클러스터 = 배포·콘솔 / 프로세스(3건)**. 실제 게임 로직 버그(1건)보다 **Play Console 제출 게이트와 서비스계정 권한**에서 마찰이 컸다. → 신규 앱 첫 제출 체크리스트를 표준화하면 가장 큰 효율 이득.
- 콘솔 게이트 3종은 증상이 비슷("제출/커밋이 막힘")하나 원인이 전부 다름(권한 / AD_ID 선언 / 등재정보 오탐). **증상이 아니라 "문제 보기" 상세로 원인을 특정**해야 오진(예: 403을 설문순서로 착각)을 피한다.
- 환경 이슈 2건 모두 뿌리는 **한글 경로 · 공유 리소스**. Windows + 한글 사용자 환경의 구조적 리스크.

## 열린 항목 (TODO / 관찰)
- [ ] **AdMob 실 유닛 미발급** — 3게임 모두 v1.0은 Google 공식 **테스트 광고 ID**로 출시(의도적). 광고는 뜨나 **수익 0**. → v1.0.1에서 AdMob 콘솔 앱 등록 후 실 ID 3종 교체 · 재배포 · 등재정보 연결 필요.
- [ ] 심사 **통과/반려 결과**를 각 게임별로 이 로그에 후속 기록.

## 관련 공통 패턴

- [[windows-ascii-tool-paths]]
- [[isolate-local-origins-by-project]]
- [[evidence-before-agent-claims]]
- [[validate-in-target-runtime]]
- [[symptom-is-not-root-cause]]

> 과거 노트에 남아 있는 `~/.claude/.../memory/` 식별자는 역사적 참조로만 보존한다. 새 기록은 특정 AI의 로컬 메모리를 기준 정보로 사용하지 않는다.
