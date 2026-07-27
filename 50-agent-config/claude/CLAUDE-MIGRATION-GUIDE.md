---
document_type: ai-migration-guide
target: claude-code
version: 1
date: 2026-07-27
---

# Claude Code 컨텍스트 체계 마이그레이션 가이드

## 목표

Claude Code의 로컬 자동 메모리를 다른 AI가 볼 수 없는 기준 정보로 사용하지 않고, 모든 AI가 읽을 수 있는 프로젝트 문서와 중앙 Obsidian Vault를 사용한다.

## Claude Code의 실제 설정 표면

- 전역 지침: `C:\Users\jihon\.claude\CLAUDE.md`
- 프로젝트 지침: 프로젝트 루트의 `CLAUDE.md` 또는 `.claude/CLAUDE.md`
- 개인 프로젝트 지침: `CLAUDE.local.md`
- 프로젝트 규칙: `.claude/rules/*.md`
- Claude Code는 `CLAUDE.md`의 `@AGENTS.md` 가져오기를 지원한다.
- 자동 메모리는 Claude Code 로컬 저장소이므로 프로젝트의 유일한 기준으로 사용하지 않는다.

공식 참고: https://code.claude.com/docs/en/memory

## 마이그레이션 원칙

1. 기존 파일을 먼저 읽고 덮어쓰지 않는다.
2. 변경 전 같은 폴더에 날짜·시간이 포함된 백업을 만든다.
3. 전역에는 개인 선호와 공통 운영 절차만 둔다.
4. 프로젝트별 사실과 현재 상태는 해당 저장소에 둔다.
5. 검증된 재사용 지식만 중앙 Vault에 승격한다.
6. 기존 설정과 충돌하면 자동 선택하지 않고 보고한다.

## 1단계: 전역 설정 점검

다음을 읽는다.

- `C:\Users\jihon\.claude\CLAUDE.md`
- `C:\Users\jihon\.claude\settings.json`
- 현재 프로젝트의 `CLAUDE.md`, `CLAUDE.local.md`
- 현재 프로젝트의 `.claude/rules/`, `.claude/settings.json`

설정 파일에 토큰이나 인증정보가 있으면 내용을 보고서에 복사하지 않는다.

## 2단계: 전역 CLAUDE.md 병합

기존 내용을 유지하고 다음 관리 구간을 하나만 둔다.

```md
<!-- shared-ai-context:start -->
# Shared AI Context Protocol

- 기본 응답 언어는 한국어다.
- 작업 전에 프로젝트 지침, 현재 Git 상태, 기존 미커밋 변경을 확인한다.
- 실제 코드, Git diff, 테스트와 실환경 증거를 AI의 이전 대화나 자기평가보다 우선한다.
- 다른 AI나 사용자의 변경을 임의로 덮어쓰지 않는다.
- 현재 작업 인계는 프로젝트의 `docs/ai/NOW.md`에 100줄 이하로 유지한다.
- 프로젝트별 사실은 전역 메모리에 넣지 않는다.
- 검증된 중요 사건과 재사용 가능한 교훈만 아래 Vault에 기록한다.
- Vault: `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
- Vault 사용 전 `00-home/WRITING-POLICY.md`를 읽는다.
- 평범한 세션이나 대화 전문은 영구 노트로 만들지 않는다.
- 비밀키, 토큰, 쿠키, 개인정보를 문서와 Git에 기록하지 않는다.
- Git 작업은 Vault의 `40-profile/GIT-POLICY.md`를 따른다.
- Push, Pull, Merge, Rebase, 브랜치 삭제는 사용자 확인 후 실행한다.
- Force push와 광범위한 변경 폐기는 금지한다.
<!-- shared-ai-context:end -->
```

이미 같은 목적의 규칙이 있다면 중복시키지 말고 한 구간으로 통합한다.

## 3단계: 현재 프로젝트 마이그레이션

먼저 저장소 루트와 기존 문서를 확인한다.

```text
AGENTS.md
CLAUDE.md
README.md
docs/
.claude/
.cursor/
.codex/
```

### 기준 구조

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

### AGENTS.md

모든 AI가 공유할 프로젝트 규칙의 기준 문서다.

- 프로젝트 목적과 중요한 경로
- 실행, 빌드, 테스트 명령
- 코드 작성 규칙
- 검증 완료 조건
- 금지 사항
- 작업 시작·인계 절차
- Git 권한

기존 `CLAUDE.md`의 공통 프로젝트 규칙은 검토 후 `AGENTS.md`로 옮긴다. Claude 전용 명령과 도구 규칙은 `CLAUDE.md`에 남긴다.

### CLAUDE.md

루트 `CLAUDE.md`에 다음 가져오기가 한 번만 존재해야 한다.

```md
@AGENTS.md
```

기존 Claude 전용 내용이 있다면 가져오기 아래에 유지한다. Windows에서는 symlink보다 가져오기를 사용한다.

### docs/ai/PROJECT.md

변하지 않는 프로젝트 지도만 기록한다.

- 한 줄 목적
- 기술 스택
- 주요 진입점
- 디렉터리 구조
- 외부 시스템
- 핵심 불변조건
- 자세한 문서의 링크

### docs/ai/NOW.md

현재 하나의 작업만 기록한다.

- 갱신 시각과 작업 AI
- 목표와 상태
- 브랜치와 HEAD 커밋
- 수정 파일
- 실행한 검증과 결과
- 미해결 문제
- 다음 AI가 실행할 정확한 다음 행동

100줄 이하로 유지하고 오래된 상태는 교체한다.

### docs/ai/BACKLOG.md

아직 시작하지 않은 작업만 우선순위와 함께 기록한다. GitHub Issues 등 이미 기준 시스템이 있다면 중복하지 않고 링크만 둔다.

### docs/ai/decisions/

되돌리기 어렵거나 여러 파일·서비스에 영향을 주는 설계 결정만 ADR로 기록한다.

## 4단계: Vault 연동

Vault 경로:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

작업 전에 다음을 읽는다.

- `00-home/WRITING-POLICY.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- 해당 프로젝트의 `20-projects/<project-id>/PROJECT-CARD.md`

새 기록은 다음 조건일 때만 만든다.

- 새로운 근본원인이 검증됨
- 반복 문제 발견
- 중요한 마일스톤 완료
- 프로젝트 단계 회고
- 두 프로젝트 이상에서 재사용할 교훈
- AI 검증 실패나 증거 불일치

아직 검증되지 않은 세션 기록은 `10-inbox/claude-code/`에 두고 `status: pending-review`를 표시한다.

## 5단계: Git 안전 정책

- 시작 전에 `git status`, 현재 브랜치, 최근 커밋을 확인한다.
- 사용자 변경과 관련 없는 파일은 커밋하지 않는다.
- 로컬 브랜치·커밋을 만들었다면 이름과 해시를 보고한다.
- Push, Pull, Merge, Rebase, 브랜치 삭제 전 사용자 확인을 받는다.
- 충돌을 자동으로 한쪽 내용으로 해결하지 않는다.
- 비밀정보를 커밋하지 않는다.

## 6단계: 검증

1. 변경 전 백업이 존재하는지 확인한다.
2. Claude Code 새 세션을 프로젝트 루트에서 시작한다.
3. `/context`로 `CLAUDE.md`가 로드되었는지 확인한다.
4. Claude에게 공통 Git 정책과 `NOW.md` 위치를 요약하게 한다.
5. `git diff`로 프로젝트 변경 범위를 확인한다.
6. 문서 링크와 Vault 경로가 실제로 존재하는지 확인한다.
7. 중복·충돌 규칙과 비밀정보가 없는지 검사한다.

## 완료 보고 형식

- 백업 위치
- 수정·생성 파일
- 기존 규칙과 병합한 내용
- 충돌 또는 보류 항목
- 현재 프로젝트 구조
- 검증 결과
- 사용자가 직접 해야 할 작업

설정 권한 때문에 진행하지 못한 경우 다른 위치에 임의 복제하지 말고 정확한 차단 경로와 필요한 권한만 보고한다.
