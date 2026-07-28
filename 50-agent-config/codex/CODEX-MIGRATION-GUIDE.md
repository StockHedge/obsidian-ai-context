---
document_type: ai-migration-guide
target: codex
version: 2
policy_version: 3
date: 2026-07-29
---

# Codex 컨텍스트 체계 마이그레이션 가이드 v2

## 목표

Codex의 한 대화나 메모리 대신 계층적 `AGENTS.md`, 프로젝트 `docs/ai/`, 중앙 Vault를 검증 가능한 공용 인계면으로 사용한다.

## 0단계: 변경 전 안전 점검

Vault 경로:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

1. Vault가 로컬 Git 저장소인지 확인한다.
2. 현재 브랜치와 변경 파일을 확인한다.
3. 미커밋 변경의 작성자와 범위를 구분하고 같은 파일을 덮어쓰지 않는다.
4. 비밀정보가 추적되거나 출력되지 않는지 확인한다.
5. 원격 생성, Push, Pull, Merge, Rebase는 실행하지 않는다.

기준선이 없거나 충돌 범위를 안전하게 분리할 수 없으면 쓰기를 중단하고 보고한다.

## Codex 설정 표면

- 전역 지침: `C:\Users\jihon\.codex\AGENTS.md`
- 프로젝트 지침: 저장소 루트와 하위 경로의 `AGENTS.md`
- 개인 설정: `C:\Users\jihon\.codex\config.toml`
- 프로젝트 설정: `.codex/config.toml`
- 더 가까운 하위 `AGENTS.md`가 해당 하위 트리에 구체적으로 적용됨

## 1단계: 기존 상태와 백업

- 전역 `AGENTS.md`, `config.toml`
- 현재 저장소와 상위·하위의 적용 가능한 `AGENTS.md`
- `.codex/config.toml`, `CLAUDE.md`, `GEMINI.md`, Cursor Rules
- README, architecture, handoff, 이슈 기준 문서
- 프로젝트 Git 상태, 브랜치, 최근 커밋

수정 대상은 타임스탬프 백업을 만든다. 관련 없는 사용자 변경을 수정하거나 커밋하지 않는다.

## 2단계: 전역 관리 블록 병합

전역 `AGENTS.md`에 다음 블록을 한 번만 둔다.

```md
<!-- shared-ai-context:start -->
# Shared AI Context Protocol

- Managed policy version: 3
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
- Git 작업은 `40-profile/GIT-POLICY.md`(policy_version 3)를 따른다.
- 작업 브랜치의 로컬 커밋, `push`, `pull --rebase`는 사용자 확인 없이 수행한다.
  사용자는 Git 명령을 직접 실행하지 않으므로 확인 요구는 미푸시 누적으로만 귀결된다.
- 보호 브랜치(`main`/`master`/`release/*`/`prod`) push, merge, 수동 rebase, cherry-pick,
  브랜치·태그 삭제, 원격 저장소 생성·변경, public 저장소 push는 사용자 확인 후 실행한다.
- Force push와 광범위한 변경 폐기는 금지한다. 훅에도 동일하게 적용된다.
- 준수는 지시가 아니라 훅으로 강제한다. Stop 훅 `git-autosync.ps1`이 `.git/autosync`
  마커가 있는 저장소에서 조건부 자동 커밋·푸시를 수행하고, SessionStart 훅
  `git-session-start.ps1`이 fetch 후 실제 Git 상태를 컨텍스트에 주입한다.
- 세션은 대체로 정상 종료되지 않는다(창 닫기, 크래시, 컨텍스트 압축).
  "세션 종료 시 정리한다"에 의존하는 규칙은 설계하지 않는다.
<!-- shared-ai-context:end -->
```

동일 목적의 기존 규칙은 중복하지 않고 병합한다. Codex 모델·샌드박스·도구 설정은 필요할 때 `config.toml`에 두고 공통 프로젝트 설명을 복제하지 않는다.

## 3단계: 프로젝트 기준 문서

```text
project/
├─ AGENTS.md
└─ docs/
   └─ ai/
      ├─ PROJECT.md
      ├─ NOW.md
      ├─ BACKLOG.md
      └─ decisions/
```

- `AGENTS.md`: 목적, 주요 경로, 실행·테스트 명령, 불변조건, 금지사항, 완료 정의, 인계, Git 권한
- `PROJECT.md`: 안정적인 구조 지도
- `NOW.md`: 현재 한 작업, 브랜치·HEAD, 변경 파일, 검증 결과, 차단점, 정확한 다음 행동
- `BACKLOG.md`: 미착수 작업 또는 이슈 트래커 링크
- `decisions/`: 장기 설계 결정의 ADR

기존에 같은 역할의 문서가 있으면 새 파일을 만들기보다 기준을 선택해 링크한다. 하위 디렉터리에 별도 규칙이 필요할 때만 더 가까운 `AGENTS.md`를 둔다.

다른 도구 어댑터는 기존 규칙을 이해한 범위에서만 갱신한다.

- `CLAUDE.md`: `@AGENTS.md` 가져오기와 Claude 전용 규칙
- `GEMINI.md`: 공통 규칙 링크와 Gemini 전용 규칙
- `.cursor/rules/`: `AGENTS.md`와 `NOW.md`를 읽게 하는 얇은 어댑터

## 4단계: Vault 기록

필수 확인:

- `00-home/WRITING-POLICY.md`
- `00-home/SYNC-BOUNDARIES.md`
- `10-inbox/INBOX-REVIEW.md`
- `40-profile/WORK-PREFERENCES.md`
- `40-profile/GIT-POLICY.md`
- 해당 프로젝트 카드

검증 전 초안은 `10-inbox/codex/`, 프로젝트 사건·마일스톤은 `20-projects/`, 일반화된 교훈은 `30-patterns/`에 둔다. 프로젝트 현재 상태를 Vault에 복제하지 않는다.

## 5단계: 검증과 배포 기록

1. 백업과 변경 diff를 확인한다.
2. 관리 블록이 정확히 한 번 있고 `Managed policy version: 2`인지 확인한다.
3. 새 Codex 작업에서 적용되는 `AGENTS.md`, `NOW.md`, Git 정책을 확인한다.
4. 링크, 경로, 중복 규칙, 비밀정보를 검사한다.
5. 실제 명령을 실행할 수 있으면 문서의 테스트·lint 명령을 검증한다.
6. Vault 변경은 검증 후 로컬 커밋한다.
7. `50-agent-config/SYNC-STATUS.md`에 실제 반영 결과를 기록한다.

완료 보고에는 백업, 변경 파일, 병합 내용, 적용된 지침 범위, Git 상태, 검증 결과, 보류 항목을 포함한다.
