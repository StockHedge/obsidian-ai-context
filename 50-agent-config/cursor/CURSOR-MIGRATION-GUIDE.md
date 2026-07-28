---
document_type: ai-migration-guide
target: cursor
version: 2
policy_version: 3
date: 2026-07-29
---

# Cursor 컨텍스트 체계 마이그레이션 가이드 v2

## 목표

Cursor User Rules와 Project Rules를 공통 `AGENTS.md`·프로젝트 `docs/ai/`·중앙 Vault에 연결하고 대화 메모리를 유일한 기준으로 사용하지 않는다.

## 0단계: 변경 전 안전 점검

Vault 경로:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

1. Vault가 로컬 Git 저장소인지 확인한다.
2. 현재 브랜치와 미커밋 변경을 확인한다.
3. 다른 Cursor 세션이나 AI가 같은 파일을 쓰는지 확인한다.
4. 변경 소유권을 구분할 수 없으면 같은 파일을 수정하지 않는다.
5. 원격 생성, Push, Pull, Merge, Rebase는 실행하지 않는다.

## Cursor 설정 표면

- 전역 지침: Cursor Settings → Rules → User Rules
- 프로젝트 규칙: `.cursor/rules/*.mdc`
- 레거시 규칙: `.cursorrules`
- 명령: `.cursor/commands/`
- Cursor CLI가 읽는 프로젝트 `AGENTS.md`, `CLAUDE.md`

Cursor Agent가 User Rules를 안전하게 직접 편집할 수 있다고 가정하지 않는다. 접근할 수 없으면 적용했다고 주장하지 않고 사용자가 붙여넣을 블록을 출력한다.

## 1단계: 기존 상태와 백업

- 현재 User Rules
- `.cursor/rules/`, `.cursorrules`, `.cursor/commands/`
- `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`
- README, architecture, handoff 문서
- 프로젝트 Git 상태, 브랜치, 최근 커밋

프로젝트 파일은 수정 전 타임스탬프 백업을 만든다. 레거시 규칙을 확인 없이 삭제하지 않는다.

## 2단계: User Rules 관리 블록

기존 User Rules와 중복되지 않게 다음 내용을 한 번만 병합한다.

```text
[Shared AI Context Protocol]
Managed policy version: 3
Canonical source: C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING\00-home\POLICY-DISTRIBUTION.md
이 관리 블록은 배포본이다. 직접 수정하지 않고 Vault 기준 문서를 먼저 수정한다.
기본 응답 언어는 한국어다.
작업 시작·재개 시 프로젝트 지침, docs/ai/NOW.md, Git 상태, 미커밋 변경을 다시 확인한다.
코드, Git diff, 테스트와 실환경 증거를 이전 대화·메모리·AI 자기평가보다 우선한다.
다른 AI나 사용자의 변경을 임의로 덮어쓰지 않는다.
현재 인계는 프로젝트 docs/ai/NOW.md에 100줄 이하로 유지한다.
프로젝트별 사실은 User Rules나 개인 메모리에 넣지 않는다.
중앙 Vault는 C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING 이다.
Vault 기록 전 00-home/WRITING-POLICY.md와 10-inbox/INBOX-REVIEW.md를 읽는다.
검증된 중요 사건과 재사용 가능한 교훈만 Vault에 승격한다.
매주 첫 세션, review_by 도래, Inbox 5건 누적, 단계 종료 시 Inbox 검토 필요를 알린다.
로컬 파일 변경이 다른 진행 중 대화에 자동 주입된다고 가정하지 않는다.
비밀키, 토큰, 쿠키, 개인정보를 문서와 Git에 기록하지 않는다.
Git 작업은 40-profile/GIT-POLICY.md(policy_version 3)를 따른다.
작업 브랜치의 로컬 커밋, push, pull --rebase는 사용자 확인 없이 수행한다.
보호 브랜치(main/master/release/*/prod) push, merge, 수동 rebase, cherry-pick, 브랜치·태그 삭제, 원격 저장소 생성·변경, public 저장소 push는 사용자 확인 후 실행한다.
Force push와 광범위한 변경 폐기는 금지한다.
세션은 대체로 정상 종료되지 않는다. "세션 종료 시 정리한다"에 의존하는 규칙은 설계하지 않는다.
[/Shared AI Context Protocol]
```

## 3단계: 프로젝트 기준 문서

```text
project/
├─ AGENTS.md
├─ .cursor/
│  └─ rules/
│     └─ shared-context.mdc
└─ docs/
   └─ ai/
      ├─ PROJECT.md
      ├─ NOW.md
      ├─ BACKLOG.md
      └─ decisions/
```

기존 문서가 같은 역할을 하면 새 파일을 만들지 않고 기준을 선택해 링크한다.

### `.cursor/rules/shared-context.mdc`

```md
---
description: Shared project context and handoff protocol
globs:
alwaysApply: true
---

Follow @AGENTS.md.

Before changing code:
- Read @docs/ai/PROJECT.md and @docs/ai/NOW.md.
- Inspect the actual Git status and existing changes.
- Treat code, Git, tests, and runtime evidence as authoritative.

Before a material handoff:
- Update @docs/ai/NOW.md with files changed, validation results, blockers, and the exact next action.
- Do not write routine session transcripts to the central Vault.
```

`AGENTS.md`는 도구 중립적인 실행·검증·아키텍처·Git 규칙을 담는다. `.cursorrules`는 공통 규칙을 새 `.mdc`로 옮긴 뒤 남은 전용 규칙과 충돌을 검토하기 전에는 삭제하지 않는다.

## 4단계: Vault 기록

먼저 읽는다.

- `00-home/WRITING-POLICY.md`
- `00-home/SYNC-BOUNDARIES.md`
- `10-inbox/INBOX-REVIEW.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- 해당 프로젝트 카드

검증 전 초안은 `10-inbox/cursor/`, 프로젝트 사건·마일스톤은 `20-projects/`, 일반화된 교훈은 `30-patterns/`에 둔다. Cursor 메모리와 체크포인트는 Git과 기준 문서를 대체하지 않는다.

## 5단계: 검증과 배포 기록

1. 백업과 프로젝트 diff를 확인한다.
2. User Rules의 관리 블록이 한 번이고 `Managed policy version: 3`인지 확인한다.
3. `.cursor/rules/shared-context.mdc`가 Cursor Rules에 표시되는지 확인한다.
4. 새 Agent 채팅에서 `AGENTS.md`, `NOW.md`, Git 정책을 확인한다.
5. `.cursorrules`와 새 Rules의 충돌, 링크, 비밀정보를 검사한다.
6. Vault 변경은 검증 후 로컬 커밋한다.
7. `50-agent-config/SYNC-STATUS.md`에 자동·수동 반영 상태를 정확히 기록한다.

완료 보고에는 백업, 변경 파일, User Rules 반영 여부, 레거시 규칙 처리, Git 상태, 검증 결과, 수동 작업을 포함한다.
