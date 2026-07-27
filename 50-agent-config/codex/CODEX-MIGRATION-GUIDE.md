---
document_type: ai-migration-guide
target: codex
version: 1
date: 2026-07-27
---

# Codex 컨텍스트 체계 마이그레이션 가이드

## 목표

Codex의 대화나 로컬 메모리를 프로젝트 기준 정보로 사용하지 않고, 계층적 `AGENTS.md`, 프로젝트 `docs/ai/`, 중앙 Obsidian Vault로 지속 가능한 인계 체계를 만든다.

## Codex의 실제 설정 표면

- 전역 지침: `C:\Users\jihon\.codex\AGENTS.md`
- 프로젝트 지침: 저장소 루트와 하위 경로의 `AGENTS.md`
- 프로젝트 설정: `.codex/config.toml`
- 개인 설정: `C:\Users\jihon\.codex\config.toml`
- 가까운 하위 `AGENTS.md`가 해당 하위 트리에서 더 구체적인 규칙으로 적용된다.
- 필수 프로젝트 규칙은 메모리가 아니라 `AGENTS.md` 또는 체크인 문서에 둔다.

공식 참고:

- https://learn.chatgpt.com/docs/agent-configuration/agents-md
- https://learn.chatgpt.com/docs/customization/memories

## 마이그레이션 원칙

1. 기존 설정과 사용자 변경을 먼저 확인한다.
2. 변경 전 타임스탬프 백업을 만든다.
3. 전역 `AGENTS.md`에는 개인 선호와 공통 프로토콜만 둔다.
4. 프로젝트 `AGENTS.md`에는 해당 저장소의 실행·검증 규칙을 둔다.
5. `.codex/config.toml`에는 Codex 설정만 두고 프로젝트 설명을 복제하지 않는다.
6. 메모리는 보조 회상 계층이며 유일한 기준이 아니다.
7. 외부 상태를 바꾸는 Git 작업은 권한 정책을 따른다.

## 1단계: 기존 상태 점검

다음을 읽는다.

- `C:\Users\jihon\.codex\AGENTS.md`
- `C:\Users\jihon\.codex\config.toml`
- 현재 저장소 루트와 상위·하위의 `AGENTS.md`
- 현재 저장소의 `.codex/config.toml`
- `CLAUDE.md`, `GEMINI.md`, `.cursor/rules/`, `.cursorrules`
- `README.md`, 기존 handoff·architecture·task 문서
- `git status`, 현재 브랜치, 최근 커밋

관련 없는 사용자 변경은 수정하거나 커밋하지 않는다.

## 2단계: 전역 AGENTS.md 병합

기존 내용을 보존하고 다음 관리 구간을 하나만 둔다.

```md
<!-- shared-ai-context:start -->
# Shared AI Context Protocol

- 기본 응답 언어는 한국어다.
- 작업 전에 적용되는 프로젝트 지침, Git 상태, 기존 미커밋 변경을 확인한다.
- 실제 코드, Git diff, 테스트와 실환경 증거를 AI의 이전 대화나 메모리보다 우선한다.
- 다른 AI나 사용자의 변경을 임의로 덮어쓰지 않는다.
- 현재 작업 인계는 프로젝트 `docs/ai/NOW.md`에 100줄 이하로 유지한다.
- 프로젝트별 사실은 전역 지침이나 개인 메모리에 넣지 않는다.
- 중앙 Vault: `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
- Vault에 쓰기 전에 `00-home/WRITING-POLICY.md`를 읽는다.
- 검증된 중요 사건과 재사용 가능한 교훈만 Vault에 승격한다.
- 평범한 세션과 대화 전문은 영구 노트로 만들지 않는다.
- 비밀키, 토큰, 쿠키, 개인정보를 문서와 Git에 기록하지 않는다.
- Git 작업은 Vault의 `40-profile/GIT-POLICY.md`를 따른다.
- Push, Pull, Merge, Rebase, 브랜치 삭제는 사용자 확인 후 실행한다.
- Force push와 광범위한 변경 폐기는 금지한다.
<!-- shared-ai-context:end -->
```

동일 목적의 기존 규칙이 있으면 중복시키지 않고 통합한다.

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

기존에 같은 역할의 문서가 있다면 새 문서를 만들기보다 기준 문서를 선택하고 링크한다.

### 프로젝트 AGENTS.md

다음을 짧고 검증 가능하게 기록한다.

- 저장소 목적과 중요한 디렉터리
- 설치, 실행, 빌드, 테스트, lint 명령
- 코드·아키텍처 불변조건
- 금지 사항
- 완료 정의
- 작업 시작과 인계 절차
- Git 권한
- `docs/ai/PROJECT.md`와 `NOW.md`를 읽어야 하는 시점

Codex 전용 설정이나 모델 선택은 `AGENTS.md`가 아니라 필요한 경우 `.codex/config.toml`에 둔다.

### docs/ai/PROJECT.md

프로젝트의 안정적인 지도만 기록한다. README와 중복되는 설명은 복사하지 말고 링크한다.

### docs/ai/NOW.md

현재 한 작업의 목표, 상태, 브랜치, 변경 파일, 검증 결과, 미해결 문제, 정확한 다음 행동을 기록한다. 100줄 이하로 유지한다.

### docs/ai/BACKLOG.md

아직 시작하지 않은 작업을 기록한다. 기존 이슈 트래커가 있으면 링크만 둔다.

### docs/ai/decisions/

장기적 구조 결정만 ADR로 기록한다.

### 다른 도구 어댑터

- `CLAUDE.md`: `@AGENTS.md`를 가져오되 기존 Claude 전용 규칙 보존
- `GEMINI.md`: `@AGENTS.md`를 가져오되 기존 Gemini 전용 규칙 보존
- `.cursor/rules/shared-context.mdc`: Cursor Agent가 `AGENTS.md`와 `NOW.md`를 읽게 함

다른 도구의 기존 설정을 완전히 이해하지 못하면 자동 재작성하지 않고 필요한 제안만 보고한다.

## 4단계: Vault 사용

Vault 경로:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

필수 확인:

- `00-home/WRITING-POLICY.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- `40-profile/PROJECT-INDEX.md`
- 해당 프로젝트 카드

검증 전 초안은 `10-inbox/codex/`, 검증된 사건·마일스톤은 `20-projects/`, 일반화된 교훈은 `30-patterns/`에 둔다.

프로젝트 현재 상태를 Vault에 복제하지 않는다.

## 5단계: Git 안전 정책

- 읽기 전용 Git 점검은 자유롭게 수행한다.
- 요청 범위 안에서 로컬 작업 브랜치와 논리적 체크포인트 커밋은 만들 수 있다.
- 브랜치와 커밋을 만들었다면 이름, 해시, 포함 파일을 보고한다.
- Push, Pull, Merge, Rebase, 삭제는 사용자 확인 전 실행하지 않는다.
- Force push, hard reset, 사용자 변경 폐기는 금지한다.
- 충돌은 자동으로 한쪽을 선택하지 않는다.

## 6단계: 검증

1. 백업 파일을 확인한다.
2. 전역과 프로젝트 `AGENTS.md`에 관리 구간이 한 번만 있는지 확인한다.
3. 현재 저장소 루트에서 새 Codex 세션을 시작한다.
4. Codex에게 적용 중인 Git 정책, `NOW.md`, Vault 경로를 요약하게 한다.
5. `git diff`와 생성 파일 목록을 검토한다.
6. 기존 테스트·lint 명령을 실행할 수 있다면 문서에 적힌 명령이 실제로 유효한지 확인한다.
7. 비밀정보, 절대경로 오기, 끊어진 링크를 검사한다.

## 완료 보고 형식

- 백업 위치
- 수정·생성 파일
- 기존 규칙과 병합한 내용
- 발견한 충돌
- 보류한 다른 도구 설정
- Git 상태
- 검증 결과
- 사용자 수동 작업

프로젝트 루트나 설정 범위를 확정할 수 없으면 추측해 새 구조를 만들지 말고 확인된 범위까지만 수행한다.
