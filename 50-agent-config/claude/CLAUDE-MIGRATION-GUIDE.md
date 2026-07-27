---
document_type: ai-migration-guide
target: claude-code
version: 2
policy_version: 2
date: 2026-07-27
---

# Claude Code 컨텍스트 체계 마이그레이션 가이드 v2

## 목표

Claude Code의 로컬 메모리나 한 대화에 프로젝트 상태를 가두지 않고, 프로젝트 저장소와 중앙 Vault를 여러 AI가 함께 읽는 기준으로 만든다.

## 0단계: 변경 전 안전 점검

중앙 Vault:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

1. Vault가 로컬 Git 저장소인지 확인한다.
2. `main`과 현재 변경 상태를 확인한다.
3. 미커밋 변경이 있으면 작성자와 범위를 구분한다. 같은 문서를 덮어쓰지 않는다.
4. Vault와 전역 설정에서 비밀정보를 출력하거나 복사하지 않는다.
5. 원격 생성, Push, Pull, Merge, Rebase는 실행하지 않는다.

Vault가 Git 저장소가 아니거나 변경 충돌의 소유권을 구분할 수 없으면 마이그레이션을 멈추고 보고한다.

## Claude Code 설정 표면

- 전역 지침: `C:\Users\jihon\.claude\CLAUDE.md`
- 프로젝트 지침: 프로젝트 루트의 `CLAUDE.md` 또는 `.claude/CLAUDE.md`
- 개인 프로젝트 지침: `CLAUDE.local.md`
- 프로젝트 규칙: `.claude/rules/*.md`
- 자동 메모리: 보조 회상 수단이며 프로젝트 기준 정보가 아님

## 1단계: 기존 상태와 백업

- 전역 `CLAUDE.md`, `settings.json`
- 프로젝트의 `CLAUDE.md`, `CLAUDE.local.md`, `.claude/`
- `AGENTS.md`, `README.md`, 기존 handoff·architecture 문서
- 프로젝트 Git 상태, 브랜치, 최근 커밋

수정할 파일은 타임스탬프 백업을 만든다. 인증정보 값은 보고서에 쓰지 않는다.

## 2단계: 전역 관리 블록 병합

다음 블록을 기존 전역 `CLAUDE.md`에 한 번만 둔다. 기존 사용자 규칙은 보존하고 같은 목적의 중복만 통합한다.

```md
<!-- shared-ai-context:start -->
# Shared AI Context Protocol

- Managed policy version: 2
- Canonical source: `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING\00-home\POLICY-DISTRIBUTION.md`
- 이 관리 블록은 배포본이다. 직접 수정하지 않고 Vault 기준 문서를 먼저 수정한다.
- 기본 응답 언어는 한국어다.
- 작업 시작·재개 시 프로젝트 지침, `docs/ai/NOW.md`, Git 상태, 기존 미커밋 변경을 다시 확인한다.
- 코드, Git diff, 테스트와 실환경 증거를 이전 대화·메모리·AI 자기평가보다 우선한다.
- 다른 AI나 사용자의 변경을 임의로 덮어쓰지 않는다.
- 현재 인계는 프로젝트 `docs/ai/NOW.md`에 100줄 이하로 유지한다.
- 프로젝트별 사실은 전역 지침이나 Claude 자동 메모리에 저장하지 않는다.
- 중앙 Vault는 `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`이다.
- Vault 기록 전 `00-home/WRITING-POLICY.md`와 `10-inbox/INBOX-REVIEW.md`를 읽는다.
- 검증된 중요 사건과 재사용 가능한 교훈만 Vault에 승격한다.
- 매주 첫 세션, review_by 도래, Inbox 5건 누적, 단계 종료 시 Inbox 검토 필요를 알린다.
- 로컬 파일 변경이 다른 진행 중 대화에 자동 주입된다고 가정하지 않는다.
- 비밀키, 토큰, 쿠키, 개인정보를 문서와 Git에 기록하지 않는다.
- Git 작업은 `40-profile/GIT-POLICY.md`를 따른다.
- Push, Pull, Merge, Rebase, 브랜치·태그 삭제, 원격 변경은 사용자 확인 후 실행한다.
- Force push와 광범위한 변경 폐기는 금지한다.
<!-- shared-ai-context:end -->
```

## 3단계: 프로젝트 기준 문서

기존에 같은 역할의 문서가 있으면 새로 복제하지 말고 기준 문서를 선택해 링크한다.

```text
project/
├─ AGENTS.md
├─ CLAUDE.md
└─ docs/
   └─ ai/
      ├─ PROJECT.md
      ├─ NOW.md
      ├─ BACKLOG.md
      └─ decisions/
```

- `AGENTS.md`: 모든 AI가 공유할 실행·검증·아키텍처·Git 규칙
- `CLAUDE.md`: `@AGENTS.md`를 한 번 가져오고 Claude 전용 규칙만 추가
- `PROJECT.md`: 안정적인 구조 지도와 불변조건
- `NOW.md`: 현재 한 작업, 브랜치·HEAD, 변경 파일, 검증, 차단점, 정확한 다음 행동
- `BACKLOG.md`: 미착수 작업 또는 기준 이슈 트래커 링크
- `decisions/`: 되돌리기 어려운 설계 결정의 ADR

실제 코드와 Git으로 확인하지 못한 내용은 `확인 필요`로 표시한다.

## 4단계: Vault 기록

먼저 읽는다.

- `00-home/WRITING-POLICY.md`
- `00-home/SYNC-BOUNDARIES.md`
- `10-inbox/INBOX-REVIEW.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- 해당 프로젝트 카드

검증 전 초안은 `10-inbox/claude-code/`, 프로젝트 사건·마일스톤은 `20-projects/`, 여러 프로젝트에 재사용할 교훈은 `30-patterns/`에 둔다. 초안에는 `created`, `review_by`, `agent`, `project`, `status`를 기록한다.

## 5단계: 검증과 배포 기록

1. 백업과 변경 diff를 확인한다.
2. 관리 블록이 정확히 한 번 있고 `Managed policy version: 2`인지 확인한다.
3. 새 Claude Code 세션에서 프로젝트 지침과 `NOW.md`가 로드되는지 확인한다.
4. Vault 경로와 문서 링크를 확인한다.
5. 중복 규칙과 비밀정보를 검사한다.
6. Vault 변경이 있으면 검증 후 로컬 커밋한다.
7. `50-agent-config/SYNC-STATUS.md`에 실제 반영 결과를 기록한다.

완료 보고에는 백업, 수정 파일, 병합 내용, Git 상태, 검증 결과, 보류 항목, 사용자가 직접 해야 할 일을 포함한다.
