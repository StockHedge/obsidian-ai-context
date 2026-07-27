---
type: agent-config-sync-status
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# AI 전역 설정 배포 상태

| 대상 | 기준 정책 | 템플릿 | 실제 설정 반영 | 마지막 확인 |
|---|---:|---:|---|---|
| Claude Code | 2 | 2 | 미확인 | 2026-07-27 |
| Codex | 2 | 2 | 미확인 | 2026-07-27 |
| Cursor User Rules | 2 | 2 | 반영됨 (policy_version 2, 관리 블록 1회) | 2026-07-27 |

## Cursor 배포 기록 (2026-07-27)

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
  - Claude Code / Codex 전역 설정은 아직 미반영

기준과 실제 설정의 버전이 다르면 [[POLICY-DISTRIBUTION]]에 따라 기준에서 다시 배포한다.
