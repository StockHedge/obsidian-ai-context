---
document_type: ai-migration-guide
target: cursor
version: 1
date: 2026-07-27
---

# Cursor 컨텍스트 체계 마이그레이션 가이드

## 목표

Cursor의 프로젝트 Rules와 공통 `AGENTS.md`를 연결하고, Cursor 대화 메모리를 프로젝트의 유일한 기준으로 사용하지 않는다.

## Cursor의 실제 설정 표면

- 전역 사용자 지침: Cursor Settings → Rules → User Rules
- 프로젝트 Rules: `.cursor/rules/*.mdc`
- 기존 `.cursorrules`: 지원되지만 레거시이므로 새 구조는 `.cursor/rules/`를 사용
- Cursor CLI: 프로젝트 루트의 `AGENTS.md`와 `CLAUDE.md`도 읽음
- Cursor IDE Agent에서는 항상 적용되는 Project Rule로 공통 문서를 명시적으로 연결하는 것이 안전함

공식 참고:

- https://docs.cursor.com/context/rules
- https://docs.cursor.com/en/cli/using

## 중요한 제약

Cursor Agent가 Settings의 User Rules를 직접 안전하게 편집할 수 있다고 가정하지 않는다. 접근할 수 없다면 아래 User Rules 블록을 사용자에게 제시하고 수동 붙여넣기를 요청한다.

## 마이그레이션 원칙

1. 기존 User Rules, `.cursor/rules/`, `.cursorrules`, `AGENTS.md`를 먼저 확인한다.
2. 프로젝트 파일은 수정 전 백업한다.
3. User Rules에는 개인 선호와 공통 프로토콜만 둔다.
4. 프로젝트 사실은 `AGENTS.md`와 `docs/ai/`에 둔다.
5. 프로젝트 Rules는 공통 문서를 연결하는 얇은 어댑터로 유지한다.
6. 레거시 규칙을 확인 없이 삭제하지 않는다.

## 1단계: 기존 상태 점검

- Cursor Settings의 User Rules 내용을 사용자에게 확인하거나 현재 세션에서 접근 가능한 범위로 읽는다.
- `.cursor/rules/**/*.mdc`
- `.cursorrules`
- `.cursor/commands/`
- 루트 `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`
- `README.md`, 기존 handoff·architecture 문서
- `git status`, 현재 브랜치, 최근 커밋

## 2단계: User Rules

기존 User Rules와 중복되지 않게 다음 내용을 병합한다.

```text
기본 응답 언어는 한국어로 한다.
작업 전에 프로젝트 규칙, Git 상태, 기존 미커밋 변경을 확인한다.
실제 코드, Git diff, 테스트와 실환경 증거를 이전 대화나 AI 자기평가보다 우선한다.
다른 AI나 사용자의 변경을 임의로 덮어쓰지 않는다.
현재 작업 인계는 프로젝트의 docs/ai/NOW.md에 100줄 이하로 유지한다.
프로젝트별 사실을 User Rules나 개인 메모리에 넣지 않는다.
중앙 Vault는 C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING 이다.
Vault에 쓰기 전에 00-home/WRITING-POLICY.md를 읽는다.
검증된 중요 사건과 재사용 가능한 교훈만 Vault에 승격한다.
평범한 세션과 대화 전문은 영구 노트로 만들지 않는다.
비밀키, 토큰, 쿠키, 개인정보를 문서와 Git에 기록하지 않는다.
Git 작업은 Vault의 40-profile/GIT-POLICY.md를 따른다.
Push, Pull, Merge, Rebase, 브랜치 삭제는 사용자 확인 후 실행한다.
Force push와 광범위한 변경 폐기는 금지한다.
```

Cursor Agent가 User Rules를 편집할 수 없으면 이 블록을 별도 보고하고 사용자가 Settings에서 붙여넣게 한다.

## 3단계: 현재 프로젝트 구조

```text
project/
├─ AGENTS.md
├─ CLAUDE.md
├─ GEMINI.md
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

기존 문서가 같은 역할을 한다면 새 파일을 만들지 말고 기준 문서를 선택해 링크한다.

### AGENTS.md

도구 중립적인 프로젝트 기준이다.

- 프로젝트 구조와 주요 진입점
- 설치, 실행, 빌드, 테스트 명령
- 코드와 아키텍처 규칙
- 완료 정의
- 금지 사항
- 시작·인계 절차
- Git 권한

### `.cursor/rules/shared-context.mdc`

다음 기본 형태를 사용하되 기존 Project Rules와 중복을 제거한다.

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

기존 `.cursorrules`는 즉시 삭제하지 않는다. 공통 규칙을 새 `.mdc`로 옮긴 뒤 남은 레거시 전용 규칙과 충돌 여부를 보고한다.

### docs/ai 문서

- `PROJECT.md`: 안정적인 프로젝트 지도
- `NOW.md`: 현재 한 작업과 다음 행동, 100줄 이하
- `BACKLOG.md`: 미착수 작업 또는 이슈 트래커 링크
- `decisions/`: 장기 설계 결정 ADR

## 4단계: Vault 사용

Vault 경로:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

읽어야 할 문서:

- `00-home/WRITING-POLICY.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- `40-profile/PROJECT-INDEX.md`
- 해당 프로젝트 카드

가공 전 초안은 `10-inbox/cursor/`, 검증된 프로젝트 사건과 마일스톤은 `20-projects/`, 일반화된 교훈은 `30-patterns/`에 둔다.

Cursor가 자체 대화에서 만든 Memory는 보조 정보로만 사용하며 코드와 프로젝트 문서에 반하면 폐기한다.

## 5단계: Git 안전 정책

- 읽기 전용 상태 확인은 자유롭게 수행한다.
- 요청된 범위 안의 로컬 브랜치와 체크포인트 커밋은 만들 수 있다.
- 생성한 브랜치·커밋은 보고한다.
- Push, Pull, Merge, Rebase, 삭제는 사용자 확인 후 실행한다.
- force push, hard reset, 사용자 변경 폐기는 금지한다.
- Cursor 체크포인트가 Git 백업을 대신한다고 가정하지 않는다.

## 6단계: 검증

1. 프로젝트 파일 백업을 확인한다.
2. `.cursor/rules/shared-context.mdc`가 Cursor Settings → Rules에 표시되는지 확인한다.
3. 새 Agent 채팅에서 적용되는 Rules를 확인한다.
4. Cursor에게 `AGENTS.md`, `NOW.md`, Git 정책을 요약하게 한다.
5. 프로젝트 변경 diff를 검토한다.
6. 레거시 `.cursorrules`와 새 Rules의 충돌을 확인한다.
7. User Rules 수동 반영이 남았는지 명확히 보고한다.

## 완료 보고 형식

- 백업 위치
- 생성·수정 파일
- User Rules 자동 적용 여부 또는 수동 붙여넣기 블록
- 레거시 규칙 처리
- 기존 규칙과 병합한 내용
- Git 상태
- 검증 결과
- 사용자 수동 작업
