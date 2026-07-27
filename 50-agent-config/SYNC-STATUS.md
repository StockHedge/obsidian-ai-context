---
type: agent-config-sync-status
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# AI 전역 설정 배포 상태

| 대상 | 기준 정책 | 템플릿 | 실제 설정 반영 | 마지막 확인 |
|---|---:|---:|---|---|
| Claude Code | 2 | 2 | 반영됨 (policy_version 2, 관리 블록 1회 / 저장소 미커밋) | 2026-07-27 |
| Codex | 2 | 2 | 반영됨 (policy_version 2, 관리 블록 1회) | 2026-07-27 |
| Cursor User Rules | 2 | 2 | 반영됨 (policy_version 2, 관리 블록 1회) | 2026-07-27 |
| Cursor 전역 폴더 (`~/.cursor`) | 2 | 2 | 반영됨 (`docs/ai/` + `shared-context.mdc`) | 2026-07-27 |

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
