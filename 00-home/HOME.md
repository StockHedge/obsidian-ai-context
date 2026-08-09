---
type: vault-home
schema_version: 2
policy_version: 2
updated: 2026-08-09
---

# AI Context Logging

이 Vault는 Claude Code, Codex, Cursor, Gemini, ChatGPT가 만든 기록을 AI별로 격리하지 않고 프로젝트와 재사용 지식 중심으로 연결한다.

## 정보의 기준 위치

1. 실제 코드와 Git 기록: 해당 프로젝트 저장소
2. 현재 진행 상태와 프로젝트 규칙: 저장소의 `AGENTS.md`, `docs/ai/PROJECT.md`, `docs/ai/NOW.md`
3. 검증된 사건·마일스톤·회고: 이 Vault의 `20-projects/`
4. 여러 프로젝트에서 재사용할 교훈: `30-patterns/`
5. 사용자 성향과 공통 운영 정책: `40-profile/`
6. 가공 전 AI 기록: `10-inbox/`

대화 전문이나 AI의 내부 추론은 기준 정보가 아니다.

## 빠른 이동

- [[WRITING-POLICY|작성·승격 정책]]
- [[VERSION-CONTROL|Vault 버전 관리]]
- [[POLICY-DISTRIBUTION|공통 정책 배포 규칙]]
- [[SYNC-BOUNDARIES|AI 간 동기화 범위와 한계]]
- [[VAULT-SYNC|기기 간 동기화 운용 규칙]]
- [[LAPTOP-SETUP|노트북 초기 설정 절차]]
- [[USER-MAINTENANCE|사용자가 직접 기록·갱신할 때]]
- [[10-inbox/INBOX-REVIEW|Inbox 검토 규칙]]
- [[VAULT-STATUS|Vault 현재 상태]]
- [[MIGRATION-MAP|2026-07-27 마이그레이션 매핑]]
- [[audits/2026-07-27-ai-context-migration-session-report|마이그레이션 종합 검수 보고서]]
- [[PROJECT-INDEX|프로젝트 인덱스]]
- [[USER-PROFILE|사용자 프로필]]
- [[WORK-PREFERENCES|공통 작업 선호]]
- [[GIT-POLICY|Git 운영 정책]]
- [[20-projects/game-factory/PROJECT-CARD|Game Factory]]
- [[20-projects/embolos/PROJECT-CARD|Embolos]]
- [[50-agent-config/README|AI별 설정 마이그레이션]]

## 일상 사용법

### 작업 시작

1. 프로젝트 저장소에서 `AGENTS.md`와 `docs/ai/NOW.md`를 읽는다.
2. `git status`, 현재 브랜치, 최근 커밋을 확인한다.
3. Vault의 Git 상태와 [[10-inbox/INBOX-REVIEW|Inbox 검토 트리거]]를 확인한다.
4. 필요할 때만 이 Vault에서 해당 프로젝트 사건과 공통 패턴을 검색한다.

### 작업 중

- 현재 상태는 프로젝트 저장소의 `NOW.md`에 기록한다.
- 장기적인 설계 결정은 프로젝트 저장소의 ADR에 기록한다.
- 아직 검증되지 않은 세션 메모는 `10-inbox/<agent>/`에 둔다.

### 체크포인트 또는 인계

- `NOW.md`에 수정 파일, 검증 결과, 미해결 문제, 정확한 다음 행동을 갱신한다.
- 새롭고 검증된 문제·교훈이 있을 때만 `20-projects/` 또는 `30-patterns/`에 승격한다.
- 단순 작업 완료나 대화 요약만으로 영구 노트를 만들지 않는다.

## 절대 규칙

- 비밀키, 토큰, 쿠키, 개인정보를 기록하지 않는다.
- AI의 완료 주장보다 실제 코드, Git diff, 테스트 결과, 실환경 증거를 우선한다.
- 추정과 확인을 구분한다.
- 기존 사용자 변경사항을 임의로 덮어쓰지 않는다.
- 다른 AI와 동시 작업할 때 같은 작업 폴더나 같은 노트를 동시에 수정하지 않는다.
- 공통 정책은 [[POLICY-DISTRIBUTION]]의 단방향 절차로만 배포한다.
- 로컬 파일 변경이 진행 중인 다른 AI 대화에 자동 주입된다고 가정하지 않는다.
