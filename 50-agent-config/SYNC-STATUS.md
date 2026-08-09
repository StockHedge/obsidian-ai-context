---
type: agent-config-sync-status
schema_version: 2
policy_version: 4
updated: 2026-08-09
---

# AI 전역 설정 배포 상태

| 대상 | 기준 정책 | 템플릿 | 실제 설정 반영 | 마지막 확인 |
|---|---:|---:|---|---|
| Claude Code (PC) | 4 | 4 | 반영됨 (policy_version 4, 관리 블록 1회, Stop·SessionStart 훅 등록) | 2026-08-09 |
| Claude Code (노트북) | 4 | 4 | **재배포 대상** — `claude-dotfiles` pull 후 확인 필요 | 2026-08-09 |
| Codex | 4 | 4 | **재배포 대상** (실제 설정 policy_version 2) | 2026-07-29 |
| Cursor User Rules | 4 | 4 | **재배포 대상** (실제 설정 policy_version 2, UI 수동 갱신 필요) | 2026-07-29 |
| Cursor 전역 폴더 (`~/.cursor`) | 4 | 4 | **재배포 대상** (`shared-context.mdc` 미갱신) | 2026-07-29 |

## v4 배포 기록 — Claude Code / PC (2026-08-09)

- 기준 문서 선행 수정: [[GIT-POLICY]] policy_version 3 → 4 (「Vault 적용」에 저장소별
  자동화 소유권 명시), [[VAULT-SYNC]] 신설, [[LAPTOP-SETUP]] 신설
- 템플릿 갱신: `50-agent-config/claude/CLAUDE-MIGRATION-GUIDE.md` (관리 블록 v4)
- 실제 설정: `%USERPROFILE%\.claude\CLAUDE.md` 관리 블록 교체.
  백업 `CLAUDE.md.bak-20260809`. 관리 블록 밖의 사용자 규칙은 한 줄도 건드리지 않았고,
  변경 이력은 파일 상단 주석 v2.6에 기록했다.
- v4에서 새로 배포된 항목
  - Vault 경로를 `%USERPROFILE%` 기준으로 전환 (PC `jihon` / 노트북 `강지호`로 프로필명이 다름)
  - Vault 원격 `StockHedge/obsidian-ai-context` 및 [[VAULT-SYNC]] 참조
  - Vault 저장소에 `.git/autosync` 마커 금지 (자동 커밋 주체는 Obsidian Git 단독)
  - Vault 파일·폴더명 변경 금지 (첨부·MOC가 `[[파일명]]` 최단 형식 링크)
  - 한글 경로 대응 `core.quotepath false` / `core.longpaths true`
  - 구 Vault 2벌(`Documents\Obsidian Vault`, 노트북 `Obsidian Vault`)은 폐기 대상
- 미반영: 노트북의 `claude-dotfiles`, Codex, Cursor 3종. 각 기기·도구에서 pull 또는
  재배포가 필요하다.

## Cursor 전역 폴더 배포 기록 (2026-07-27 21:40 KST)

- 경로: `C:\Users\jihon\.cursor`
- Git: 없음
- 백업: `C:\Users\jihon\Downloads\AI-CONTEXT-MIGRATION-PACK-v2-20260727\backups\cursor-global-20260727-213936\`
- 추가:
  - `.cursor/rules/shared-context.mdc`
  - `docs/ai/PROJECT.md`, `NOW.md`, `BACKLOG.md`, `decisions/README.md`
- 갱신: `AGENTS.md` (Shared Context 절·SoT 행; 기존 본문 보존)
- User Rules 선호 규칙 id `16939507`: 갱신된 `AGENTS.md`와 재동기화 시도
- Shared Protocol id `16992538`: 그대로 유지 (중복 추가 없음)
- `.cursorrules`: 없음 → 삭제하지 않음
- Push/Pull/Merge/Rebase/원격 변경: 없음
- 수동 보류:
  - `~/.cursor`를 워크스페이스로 연 상태에서 Project Rules `shared-context` UI 표시 확인
  - Claude Code 전역 설정 미확인 항목

## Cursor 프로젝트(embolos) 배포 기록 (2026-07-27)

- 적용 `policy_version`: 2
- 실제 설정 위치: Cursor Settings → Rules → User Rules
- 규칙 id: `16992538` / title `[Shared AI Context Protocol]`
- 기존 전역 선호 규칙 id: `16939507` (유지, 덮어쓰지 않음)
- 중복 생성분 `16992532`는 즉시 제거해 관리 블록 1회만 유지
- 백업: `C:\Users\jihon\Downloads\AI-CONTEXT-MIGRATION-PACK-v2-20260727\backups\cursor-20260727-203938\`
- 프로젝트 반영: `C:\Users\jihon\projects\embolos`
  - `AGENTS.md` (도구 중립)
  - `.cursor/rules/shared-context.mdc` (얇은 어댑터)
  - `docs/ai/PROJECT.md`, `NOW.md`, `BACKLOG.md`, `decisions/`
- `.cursorrules`: 저장소에 없음 → 삭제하지 않음
- Git 원격 작업: Push/Pull/Merge/Rebase/원격 변경 없음
- 수동 보류:
  - Cursor UI에서 Project Rules에 `shared-context` 표시 확인
  - 새 Agent 채팅에서 `AGENTS.md`/`NOW.md` 재확인
  - embolos `docs/ai/`·규칙 파일 로컬 커밋 여부 (사용자 결정)

## Codex 배포 기록 (2026-07-27)

- 적용 `policy_version`: 2
- 실제 전역 설정: `C:\Users\jihon\.codex\AGENTS.md`
- 적용 범위: `<!-- shared-ai-context:start -->`부터 `<!-- shared-ai-context:end -->`까지 관리 블록 1회
- 백업:
  - 전역 `AGENTS.md` 생성 전 부재 상태와 초기 상태 파일: `C:\Users\jihon\.codex\backups\codex-v2-migration-20260727-203935-KST\`
  - Cursor 배포 커밋 후 `SYNC-STATUS.md` 재백업: `C:\Users\jihon\.codex\backups\codex-v2-migration-20260727-204435-KST\`
- 기존 `C:\Users\jihon\.codex\config.toml`: 읽기·점검만 했으며 공통 정책이나 프로젝트 사실을 추가하지 않았다.
- 검사 결과: 관리 블록 시작·끝 표식 각각 1개, `Managed policy version: 2` 1개, 기준 정책 경로·상대 Markdown 링크·비밀정보 서명 검사 통과.
- 정책 흐름: Vault 기준 문서는 수정하지 않고, 기준 → 배포본 방향으로만 전역 관리 블록을 반영했다.
- Inbox 점검: 문서 5개, `review_by` 도래·경과 문서 0개. 5개 누적 트리거가 충족되어 사용자 검토가 필요하며 자동 승격·삭제는 하지 않았다.
- 수동 보류: Vault 전체 검사에서 `20-projects/game-factory/PROJECT-CARD.md`의 기존 `\|` 포함 위키 링크 10개가 발견됐다. 대상 사고 파일은 존재하지만 링크 표기 의도는 확인이 필요하며, Codex 배포 범위와 무관해 수정하지 않았다.

## Claude Code 배포 기록 (2026-07-27 20:41~21:50 KST)

- 적용 `policy_version`: 2
- 실제 전역 설정: `C:\Users\jihon\.claude\CLAUDE.md`
  파일 최상단에 `<!-- shared-ai-context:start -->` ~ `:end` 관리 블록 1회 병합
- 대상 프로젝트: `claude-dotfiles` (`C:\Users\jihon\.claude`, `main`, HEAD `ded36a3`)
  — 이 디렉터리가 Claude Code 전역 설정 디렉터리이면서 저장소 루트다.
  카드: [[20-projects/claude-dotfiles/PROJECT-CARD]]
- 프로젝트 반영:
  - `AGENTS.md` (도구 중립, 64줄 — 전역 임포트 대상이라 짧게 유지)
  - `docs/ai/PROJECT.md`, `NOW.md`(82줄), `BACKLOG.md`,
    `docs/ai/decisions/ADR-0001-agents-import-in-global-claude-md.md`
  - `.gitignore`에 `!/AGENTS.md`, `!/docs/` 추가
    — 이 저장소는 deny-all 화이트리스트 방식이라 등록하지 않으면 신규 문서가 추적되지 않는다.
- 백업: `C:\Users\jihon\.claude\backups\ai-context-migration-v2-20260727-204110\`
  (`CLAUDE.md`, `gitignore`, `settings.json`, `README.md`, 그리고 수정 전 Vault
  `SYNC-STATUS.md`·`PROJECT-INDEX.md` 사본). 저장소 `.gitignore` 대상이라 추적되지 않는다.
- 구조 결정: 전역 `CLAUDE.md`가 `@AGENTS.md`를 임포트한다. 임포트는 조건부가 아니어서
  다른 프로젝트 세션에도 로드되며, 사용자가 그 비용을 승인했다. 근거는 저장소의 ADR-0001.
- 검사 결과
  - 관리 블록 시작·끝 표식 각 1개, `Managed policy version: 2` 1개. 중복 없음.
  - 기존 사용자 규칙은 삭제·요약 없이 전부 보존. 의미가 겹치는 항목(한국어 응답,
    검증 우선순위)은 기존 규칙이 더 구체적이라 병존시키고, 충돌 시 Vault 기준 문서가
    우선한다는 문구를 `CLAUDE.md` v2.5 이력에 명시했다.
  - 비밀정보: `.credentials.json`, `backups/`, `history.jsonl`, `projects/`, `sessions/`가
    모두 ignore 상태 유지. 자격증명 값을 문서·보고서에 기록하지 않았다.
  - 검증 실행: `settings.json` JSON 파싱 PASS, 추적 대상 `.ps1` 6종 구문 검사 PASS,
    `settings.json`이 참조하는 훅 경로 4건 실존 확인, `pre-commit` 가드 설치 확인.
- 동시 작업 관측: 이 세션(20:41 시작) 진행 중 Cursor·Codex가 Vault에 커밋
  (`b830cd3`, `77fb654`, `f9b3818`). 편집 직전 재읽기로 감지해 **덮어쓰지 않고** 병합했다.
  [[WRITING-POLICY]]의 동시 작업 항목과 [[SYNC-BOUNDARIES]]의 경고가 실제로 발생한 사례다.
- Inbox 점검: 미분류 초안 4건 (`10-inbox/cursor/` 4개), `review_by` 도래·경과 0건
  (모두 2026-08-03). "5건 누적" 트리거는 미충족이나, 오늘이 월요일이라 **"매주 첫 세션"**과
  이번 마이그레이션에 따른 **"큰 인계 직전"** 트리거가 충족된다. 사용자 검토가 필요하며
  자동 승격·삭제는 하지 않았다. (Codex 기록의 "문서 5개"는 `README.md`·`INBOX-REVIEW.md`를
  포함한 수로 보이며, 초안만 세면 당시 3건·현재 4건이다.)
- Push/Pull/Merge/Rebase/브랜치·태그 삭제/원격 변경: 없음
- 수동 보류
  1. **저장소 커밋 안 함.** `claude-dotfiles`에 이번 세션 이전의 미커밋 변경 12건이
     있어 [[GIT-POLICY]]의 "다른 AI·사용자의 미커밋 변경 임의 포함" 금지에 따라 분리했다.
     산출물만 골라 커밋하는 것은 사용자 결정이다.
  2. **새 세션 로드 확인 미완료.** 현재 세션이 편집 이전에 시작되어 `@AGENTS.md`가
     실제 인라인되는지 확인할 수 없었다.
  3. `scripts/` 추적 누락(문서-실제 불일치)은 마이그레이션 범위 밖이라 손대지 않고
     저장소 `docs/ai/BACKLOG.md` 1번에 기록했다.

기준과 실제 설정의 버전이 다르면 [[POLICY-DISTRIBUTION]]에 따라 기준에서 다시 배포한다.


## Claude Code 배포 기록 — policy_version 3 (2026-07-29 02:20~02:45 KST)

### 개정 배경

`GIT-POLICY.md` v2는 `push`·`pull`을 사용자 확인 대상으로 두었으나, 사용자는 Git 명령을
직접 실행하지 않는다. 결과적으로 확인 대기 상태가 방치로 귀결됐다.

2026-07-29 실측:

- 로컬 저장소 21개 중 미푸시 커밋 50건 (AICOMMERCE 31, game-factory 15, game-standards 3, 기타 4)
- `AIWebbuildernewver`는 원격이 로컬보다 96커밋 앞서 있었음 (`Vibecoding`은 3커밋)
- 전역 설정 저장소 `~/.claude`(claude-dotfiles) 자체가 미커밋 15건, 마지막 커밋 2026-07-10

동시에 확인된 사실은 지시문만으로는 준수가 보장되지 않는다는 것이다. `CLAUDE.md`가
`PROGRESS.md` 규칙을 담고 있던 저장소(AICOMMERCE)에서도 동일한 누락이 발생했다.
공식 문서 확인 결과 Claude Code의 `SessionEnd` 훅은 `clear`/`resume`/`logout`/
`prompt_input_exit` 등에서 발화하며 터미널 종료·크래시에서는 신뢰할 수 없다.
따라서 "세션 종료 시 정리" 형태의 규칙은 가장 필요한 순간에 실행되지 않는다.

### 기준 문서 변경

- `40-profile/GIT-POLICY.md` → `policy_version: 3`
  - 작업 브랜치의 로컬 커밋·`push`·`pull --rebase`를 확인 없이 허용
  - 보호 브랜치 정의 신설 (`main`/`master`/`release/*`/`prod`/`production`)
  - 확인 대상에 `cherry-pick`·`revert`·public 저장소 push 추가
  - 「자동화 계층」 절 신설 — opt-in 마커, Stop/SessionStart 훅, 가드 목록, 훅 실패 원칙
- `00-home/POLICY-DISTRIBUTION.md` → `policy_version: 3`

### 템플릿 변경

- `50-agent-config/claude/CLAUDE-MIGRATION-GUIDE.md` — 관리 블록 v3
- `50-agent-config/codex/CODEX-MIGRATION-GUIDE.md` — 관리 블록 v3
- `50-agent-config/cursor/CURSOR-MIGRATION-GUIDE.md` — 관리 블록 v3
  (Cursor는 `<!-- -->` 주석이 아니라 `[Shared AI Context Protocol]` 평문 구분자를 사용)

### 실제 설정 반영 (Claude Code만)

- `C:\Users\jihon\.claude\CLAUDE.md` — 관리 블록 1회, `Managed policy version: 3` 검증 완료
- `C:\Users\jihon\.claude\settings.json` — `Stop`, `SessionStart` 훅 등록.
  기존 `PostToolUse`/`TaskCompleted`/`TeammateIdle` 훅과 `statusLine`,
  `enabledPlugins` 21건, `model`, `permissions` 기존 항목 전부 보존 확인
- `permissions.allow` +13건 (git 계열), `permissions.deny` +4건
  (`git push --force`, `-f`, `--force-with-lease`, `git reset --hard`)
- 신규 훅 스크립트
  - `~/.claude/hooks/git-autosync.ps1` (Stop, timeout 120s)
  - `~/.claude/hooks/git-session-start.ps1` (SessionStart, timeout 60s)
- 백업
  - `~/.claude/backups/settings-20260729-023539.json`
  - `~/.claude/backups/policy-v3-20260729-023639/`
  - `~/.claude/backups/CURSOR-MIGRATION-GUIDE-20260729-023915.md`

### 훅 등록 형식 변경 (중요)

기존 훅은 shell form 문자열에 `$env:USERPROFILE`을 포함하고 있었다. Windows에서 훅
셸이 Git Bash일 경우 bash가 `$env`를 빈 문자열로 전개해 경로가 `:USERPROFILE\...`로
깨진다(실측 확인). 신규 훅은 exec form(`command` + `args`)으로 등록해 셸을 거치지 않으며,
`$env:USERPROFILE`이 PowerShell에 그대로 전달된다. 기존 3개 훅도 같은 형식으로 통일했다.

또한 `hooks/*.ps1` 중 한글이 포함된 파일에 UTF-8 BOM이 없어 PowerShell 5.1이 CP949로
오독하는 상태였다. `post-tool-format.ps1`을 포함해 BOM을 추가했다.

### 자동화 적용 범위 (opt-in)

`.git/autosync` 마커가 있는 저장소에서만 동작한다. 머신별 설정이며 커밋되지 않는다.

- 적용 13개: block-clear, game-factory, game-standards, mobile-claude-code-1, order-pop,
  tangle-out, AICOMMERCE, AIWebbuilder, AIWebbuildernewver, embolos, Aibetatester,
  Nunathings, Vibecoding
- 제외 8개: 원격 없는 7개(jeonnam-alarm, alchemy-bounce-master(AGY), color-gate(AGY),
  crowd-clash(AGY), ai-video-viral, imjin2-reboot, AI-CONTEXT-LOGGING),
  그리고 **malrangee — `gh repo view`로 PUBLIC 확인되어 제외**
- `~/.claude`(claude-dotfiles)와 이 Vault는 의도적으로 제외했다.
  전자는 설정 변경의 중간 상태가 자동 커밋되면 이력이 오염되고,
  후자는 「검증 후 커밋」 원칙과 충돌한다.

### 검증 결과 (스크래치 저장소 + bare origin, 9종)

1. 마커 없음 → 무동작 ✔
2. 마커 있음 + 보호 브랜치 `main` → 커밋만, push 안 함 ✔
3. 스로틀(600초) → 즉시 재실행 시 무동작 ✔
4. 작업 브랜치 `ai/test-branch` → 커밋 + `push -u`, 원격 HEAD 일치 ✔
5. `.env`에 `sk-ant-api03-…` 배치 → 커밋 중단 + `systemMessage` 경고 ✔
6. `MERGE_HEAD` 존재 → 건너뜀 ✔
7. detached HEAD → 건너뜀 ✔
8. git 저장소 아님 → 무출력 종료 ✔
9. SessionStart 훅 JSON 파싱 성공, `additionalContext` 한글 정상 ✔

### 보류 / 사용자 조치 필요

- **Codex 재배포** — `C:\Users\jihon\.codex\AGENTS.md`가 policy_version 2. 파일 기반이라
  자동 반영 가능하나 이번 범위 밖으로 두었다.
- **Cursor User Rules 재배포** — Cursor Settings → Rules → User Rules (규칙 id `16992538`).
  UI 편집이라 수동 작업이다.
- **`~/.cursor/rules/shared-context.mdc`** 미갱신.
- **Vault 원격 백업** — 이 Vault는 여전히 원격이 없다. 2026-07-29 스캔 결과
  실제 비밀값 0건, 67파일 0.2MB로 private repo 백업에 지장이 없다.
  `GIT-POLICY.md`의 "원격 연결과 Push는 사용자 승인 전에는 하지 않는다" 규정에 따라
  사용자 결정 대기.
- 훅 등록 경로는 `$env:USERPROFILE` 기반이라 머신 독립적이나, 노트북에서 Claude Code를
  쓰려면 `claude-dotfiles` 저장소를 먼저 동기화해야 한다 (현재 미커밋 15건).

### 사건 기록 — 첫 실사용 발화 (2026-07-29 02:39 KST)

배포 직후, 진행 중이던 embolos 세션에서 `Stop` 훅이 실제로 발화했다.
이는 settings.json 훅 변경이 **실행 중인 세션에도 즉시 적용된다**는 실증이기도 하다.

- 커밋 `2c01d9f` — `chore(autosync): WIP 2026-07-29 02:39 [claude-code]`
- 브랜치 `legal-kakaopay-2026-07` (작업 브랜치) → 커밋 + push 수행
- 내용: 256개 항목 (신규 231 / 수정 25). `docs/` 230건은 대부분
  `docs/handoff/cafe24_ocr/*.txt` (OCR 산출물), 나머지 `backend/` 23, `tools/` 3
- 검사 결과: 비밀값 경로 0건, 캐시·빌드 산출물 0건, `.gitignore`는 적절히 구성돼 있었음
- 유실 없음. 워킹트리는 이후 clean

판단: 훅은 설계대로 동작했고 잃은 것은 없다. 그러나 "생성된 230개 OCR 산출물이
저장소에 속하는가"는 저장소 내용에 대한 결정이며, 그 결정이 세션과 사용자 모르게
기본값으로 내려졌다. 대량 변경은 안전망의 대상이 아니라 결정의 대상이다.

조치: `git-autosync.ps1`에 blast radius 게이트 추가 (기본 40건, `CLAUDE_AUTOSYNC_MAX_FILES`
로 조정). 한도 초과 시 커밋하지 않고 경고만 낸다. `GIT-POLICY.md` 가드 목록에 반영.
검증: 10건 → 커밋 / 60건 → 보류+경고 / 한도 100 상향 후 60건 → 커밋, 3종 실행 확인.
스크립트 구문 파서 검사 0 오류, BOM 유지 확인.
