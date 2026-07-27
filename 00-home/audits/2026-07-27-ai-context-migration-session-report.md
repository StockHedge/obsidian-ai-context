---
document_type: session-audit-report
version: 1
date: 2026-07-27
status: completed
---

# AI 컨텍스트 통합·Vault 마이그레이션 종합 검수 보고서

## 1. 보고서 목적

이 문서는 여러 AI 사이의 프로젝트 컨텍스트 동기화 문제에 관한 이번 대화의 요청, 조사, 설계 판단, 실제 파일 작업, 오류와 복구, 검증 결과를 다음 AI 또는 사람이 독립적으로 검수할 수 있도록 정리한 기록이다.

## 2. 사용자 문제와 요청 경과

### 최초 문제

사용자는 Claude를 주력으로 ChatGPT, Cursor, Gemini, Codex 등 여러 AI를 사용한다. 구독·토큰·사용량 때문에 도구를 전환하지만 한 AI와 진행한 대화와 코딩 작업이 다른 AI의 메모리에 실시간 반영되지 않아, 직접 이전 AI에게 요약을 받아 다음 AI에 전달하는 비효율을 겪고 있었다.

Obsidian에 AI가 컨텍스트를 기록하도록 하는 기존 Rule을 사용하고 있었다.

### 1차 설계 요청

- 여러 AI가 공유할 수 있는 방법 제안
- Claude Code에서 Codex로 전환해도 작업을 이어갈 구조
- Obsidian 활용 방식 검토

### 2차 상세 검토 요청

- 모든 AI 전역 설정과 현재 프로젝트에 공통 프로토콜을 적용할지 판단
- 새 Obsidian Vault 직접 확인
- 세션 종료 시 코드·구조 문제, 해결법, 교훈을 기록하는 방식 검토
- Git Branch, Commit, Push, Pull 설명
- 모든 프로젝트 요약을 개인 AI 메모리에 저장할지 판단
- 프로젝트 폴더와 Markdown 역할 재설계

### 이번 실행 요청

1. 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`를 실제로 구조화하고 기존 내용을 유기적으로 이동
2. Claude, Codex, Cursor 각각에 맞는 마이그레이션 가이드와 채팅창용 프롬프트 생성
3. 이번 세션의 요청과 수행 내용을 포함한 종합 검수 문서 생성

## 3. 설계 결론

정보를 다음 네 계층으로 분리했다.

| 계층 | 기준 정보 |
|---|---|
| AI 전역 설정 | 개인 선호, 공통 작업 절차, Git 권한 |
| 프로젝트 저장소 | 실제 코드, `AGENTS.md`, `docs/ai/PROJECT.md`, `NOW.md`, ADR |
| 중앙 Obsidian Vault | 검증된 사건, 마일스톤, 회고, 교차 프로젝트 패턴 |
| AI 내장 메모리 | 사용자 성향을 회상하는 보조 계층 |

AI별 영구 폴더는 다시 사일로를 만들기 때문에 폐기하고, AI별 폴더는 가공 전 기록을 받는 `10-inbox/`로만 유지했다.

## 4. 작업 전 Vault 조사

조사 당시 Vault는 Git 저장소가 아니었다.

확인된 주요 자산:

- GameFactory 대시보드 1개
- GameFactory 사건 로그 10개
- Embolos 남은 Phase·트랙 문서
- Cursor가 작성한 Embolos AI Company Ops Console 세션 로그
- ChatGPT, Claude, CursorAI, Gemini, 기타 AI별 폴더
- Obsidian 기본 설정 폴더 `.obsidian`

기존 GameFactory 로그는 “증상 → 근본원인 → 수정 → 재발방지” 구조가 잘 잡혀 있어 본문을 보존하고 분류와 상태 속성만 보강하기로 했다.

확인된 위험:

- AI별 영구 지식 사일로
- 중복된 `AI-CONTEXT-LOGGING/AI-CONTEXT-LOGGING` 경로
- 수동 대시보드 날짜와 실제 로그 불일치
- Claude 로컬 메모리 경로에 대한 종속 참조
- 추정 원인과 확인 원인의 혼재
- 특수문자 폴더명과 자동화 마찰
- 여러 AI가 쓰지만 버전 백업 없음

## 5. 백업과 복구 가능성

기존 구조를 이동하기 전에 다음 ZIP을 생성했다.

- 파일: `AI-CONTEXT-LOGGING-pre-migration-20260727.zip`
- 크기: 23,703 bytes
- SHA-256: `8C6095D483347A1AECE9218DB649E205C1CDD25040B359ED2D2917ABFC389B68`

백업 이후 다른 Cursor 세션이 새로 추가한 문서는 ZIP에는 없지만 현재 Vault에서 별도로 감지해 보존했다.

## 6. 실제 마이그레이션 매핑

### 프로젝트 자료

| 이전 | 이후 |
|---|---|
| `PROJECTS/GameFactory-BugLog/_dashboard.md` | `20-projects/game-factory/PROJECT-CARD.md` |
| `PROJECTS/GameFactory-BugLog/logs/` | `20-projects/game-factory/incidents/` |
| `PROJECTS/Embolos/남은 Phase와 트랙.md` | `20-projects/embolos/PROJECT-CARD.md` |
| Cursor의 Embolos Ops 본편 로그 | `20-projects/embolos/milestones/2026-07-26-ai-company-ops-console.md` |

### AI별 기존 폴더

| 이전 | 이후 |
|---|---|
| `ChatGPT/` | `10-inbox/chatgpt/` |
| `Claude/` | `10-inbox/claude-code/` |
| `CursorAI/` | `10-inbox/cursor/` |
| `Gemini/` | `10-inbox/gemini/` |
| `TheOtherThings/` | `10-inbox/other/` |
| 없음 | `10-inbox/codex/` |

내용 이동 후 비어 있음을 확인한 이전 `PROJECTS/`와 중첩 `AI-CONTEXT-LOGGING/` 컨테이너만 제거했다. 내용이 있는 문서는 삭제하지 않았다.

## 7. 마이그레이션 중 외부 변경 처리

작업 중 다른 Cursor 세션이 다음 파일을 추가했다.

- 컨텍스트 로깅 경로 정정 세션 로그
- Embolos 종합 마이그레이션 핸드오프
- 해당 핸드오프 포인터

파일을 덮어쓰거나 삭제하지 않고 현재 정책에 맞게 처리했다.

- 검증된 Ops 구현 로그: Embolos 마일스톤
- 로깅 경로 정정 로그: `10-inbox/cursor/`
- 임시 Embolos 인계와 포인터: `10-inbox/cursor/`

Embolos 임시 인계는 실제 프로젝트 저장소에 상시 `docs/ai/NOW.md`가 정착될 때까지 보존하도록 `pending-project-migration` 상태를 부여했다.

## 8. 최종 Vault 구조

```text
AI-CONTEXT-LOGGING/
├─ .obsidian/
├─ 00-home/
│  ├─ HOME.md
│  ├─ WRITING-POLICY.md
│  ├─ VAULT-STATUS.md
│  ├─ MIGRATION-MAP.md
│  └─ audits/
├─ 10-inbox/
│  ├─ chatgpt/
│  ├─ claude-code/
│  ├─ codex/
│  ├─ cursor/
│  ├─ gemini/
│  └─ other/
├─ 20-projects/
│  ├─ embolos/
│  │  ├─ PROJECT-CARD.md
│  │  ├─ incidents/
│  │  ├─ milestones/
│  │  └─ retrospectives/
│  └─ game-factory/
│     ├─ PROJECT-CARD.md
│     ├─ incidents/
│     ├─ milestones/
│     └─ retrospectives/
├─ 30-patterns/
│  ├─ architecture/
│  ├─ debugging/
│  ├─ deployment/
│  ├─ environment/
│  └─ ai-reliability/
├─ 40-profile/
├─ 50-agent-config/
├─ 90-archive/
└─ _templates/
```

## 9. 새로 작성한 운영 자산

### Vault 운영

- 홈과 빠른 탐색
- 작성·승격 정책
- Vault 상태
- 마이그레이션 매핑
- Inbox, Projects, Patterns, Profile, Archive 사용 안내

### 사용자·Git 정책

- 확인된 사용자 프로필
- 공통 작업 선호
- 프로젝트 인덱스
- Git 권한과 금지 작업

### 템플릿

- Project Card
- Incident
- Pattern
- Retrospective
- Session Capture
- 프로젝트 `NOW.md`
- ADR

### 기존 사건에서 추출한 교차 프로젝트 패턴

- Windows 도구 경로는 ASCII 우선
- 로컬 웹 프로젝트 origin 분리
- AI의 완료 주장보다 독립 증거 우선
- 최종 대상 런타임의 최소 스모크 테스트
- 같은 증상이라도 세부 증거로 원인 구분

### 기존 사건 정규화

GameFactory 사건 10개에 다음 공통 속성을 보강했다.

- `type: incident`
- `schema_version: 1`
- `project`
- `component`
- `status`
- `root_cause_status`

본문과 역사적 세부사항은 보존했다.

## 10. AI별 마이그레이션 세트

### Claude Code

- `CLAUDE-MIGRATION-GUIDE.md`
- `CLAUDE-CHAT-PROMPT.md`

반영한 실정:

- 전역 `C:\Users\jihon\.claude\CLAUDE.md`
- 프로젝트 `CLAUDE.md`
- `@AGENTS.md` 가져오기
- `/context` 검증
- Claude 자동 메모리는 로컬 보조 계층

### Codex

- `CODEX-MIGRATION-GUIDE.md`
- `CODEX-CHAT-PROMPT.md`

반영한 실정:

- 전역 `C:\Users\jihon\.codex\AGENTS.md`
- 프로젝트·하위 경로의 계층적 `AGENTS.md`
- `.codex/config.toml`과 지침 문서 역할 분리
- 로컬 메모리를 필수 규칙의 기준으로 사용하지 않음

### Cursor

- `CURSOR-MIGRATION-GUIDE.md`
- `CURSOR-CHAT-PROMPT.md`

반영한 실정:

- User Rules는 Settings에서 관리
- 프로젝트 `.cursor/rules/*.mdc`
- 레거시 `.cursorrules` 보존·점진 이전
- Cursor CLI와 IDE의 지침 로딩 차이
- User Rules를 자동 편집할 수 없으면 수동 붙여넣기 블록 보고

## 11. 오류와 대응

### 백업 경로 불일치

첫 백업 시도 직전에 사용자가 기존 `★PROJECTS★`를 `PROJECTS`로 변경하고 Embolos 자료를 추가했다. 존재하지 않는 이전 경로를 감지해 중단했고 아무 파일도 이동하지 않았다. 새 현재 상태를 다시 읽고 백업 대상을 재계산했다.

### PowerShell 디렉터리 생성 매개변수

첫 구조 이동 시도에서 현재 PowerShell의 `New-Item`이 `-LiteralPath`를 지원하지 않아 디렉터리 생성과 이동이 실패했다. 기존 파일은 남아 있음을 확인했다. 이후 오류 즉시 중단 모드와 `-Path`를 사용해 경로 검증부터 다시 수행했고 이동을 완료했다.

이 두 오류는 기존 내용 손실을 일으키지 않았다.

## 12. 검증 결과

보고서 작성 직전 검증:

- Markdown UTF-8 오류: 0
- 잘못 닫힌 frontmatter: 0
- 끊어진 Obsidian Wiki 링크: 0
- 이전 최상위 `PROJECTS`, `★PROJECTS★`, 중첩 `AI-CONTEXT-LOGGING`: 0
- GameFactory 사건 파일: 10
- Vault의 AI 설정 문서: 7개 (`README` 포함)
- 출력본과 Vault의 Claude/Codex/Cursor 6개 문서 SHA-256 일치: 전부 일치

## 13. 수행하지 않은 작업

다음은 이번 요청 범위에서 의도적으로 수행하지 않았다.

- Claude, Codex, Cursor 실제 전역 설정 수정
- Embolos나 GameFactory 실제 저장소의 `AGENTS.md`, `docs/ai/` 도입
- Vault Git 저장소 초기화
- Git commit, push, pull, merge, rebase
- 클라우드 배포나 외부 서비스 변경
- Obsidian 커뮤니티 플러그인 설치

각 AI 전역 설정과 실제 프로젝트는 생성한 마이그레이션 세트를 해당 AI에 첨부해 프로젝트별로 적용해야 한다.

## 14. 권장 다음 순서

1. Claude Code에 Claude 세트를 첨부해 전역 설정과 가장 중요한 프로젝트 하나를 먼저 마이그레이션
2. 새 세션에서 `AGENTS.md`, `NOW.md`, Vault 연결이 실제로 로드되는지 검증
3. Codex 세트 적용
4. Cursor 세트 적용과 User Rules 수동 확인
5. Embolos의 임시 handoff를 프로젝트 `docs/ai/NOW.md`로 흡수
6. 다른 프로젝트로 확대
7. Vault를 비공개 Git 또는 다른 버전 백업에 연결할지 결정

## 15. 공식 동작 확인에 사용한 문서

- Claude Code Memory: https://code.claude.com/docs/en/memory
- Codex `AGENTS.md`: https://learn.chatgpt.com/docs/agent-configuration/agents-md
- OpenAI Memories: https://learn.chatgpt.com/docs/customization/memories
- Cursor Rules: https://docs.cursor.com/context/rules
- Cursor CLI: https://docs.cursor.com/en/cli/using

## 16. 최종 판정

중앙 Vault 구조화, 기존 자료 보존·이동, 공통 운영 정책, 교차 프로젝트 패턴, 세 도구별 마이그레이션 세트 생성은 완료됐다.

실제 AI 전역 설정과 개별 코드 저장소 마이그레이션은 아직 실행되지 않았으며, 각 AI 세트를 이용한 후속 단계로 남아 있다.
