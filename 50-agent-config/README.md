---
type: agent-config-index
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# AI별 설정 마이그레이션 v2

각 도구의 실제 지침 로딩 방식과 설정 위치가 다르므로 공통 문장을 무조건 복사하지 않고 도구별 문서를 사용한다.

이 디렉터리는 [[POLICY-DISTRIBUTION]]에 따른 배포 템플릿이다. 기준 정책은 `00-home/`과 `40-profile/`에 있으며 여기나 실제 전역 설정을 먼저 수정하지 않는다.

## Claude Code

- [[claude/CLAUDE-MIGRATION-GUIDE]]
- [[claude/CLAUDE-CHAT-PROMPT]]

## Codex

- [[codex/CODEX-MIGRATION-GUIDE]]
- [[codex/CODEX-CHAT-PROMPT]]

## Cursor

- [[cursor/CURSOR-MIGRATION-GUIDE]]
- [[cursor/CURSOR-CHAT-PROMPT]]

모든 마이그레이션은 Vault Git 상태를 확인하고 기존 설정을 백업한 뒤 병합한다. 프로젝트별 정보는 전역 설정에 넣지 않는다.

- [[SYNC-STATUS|실제 전역 설정 배포 상태]]
- v1 템플릿 기준선: Git 커밋 `877862c`
- 현재 템플릿 정책 버전: 2
