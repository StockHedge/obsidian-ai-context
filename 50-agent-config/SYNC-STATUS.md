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
| Codex | 2 | 2 | 반영됨 (policy_version 2, 관리 블록 1회) | 2026-07-27 |
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

## Codex 배포 기록 (2026-07-27)

- 적용 `policy_version`: 2
- 실제 전역 설정: `C:\Users\jihon\.codex\AGENTS.md`
- 적용 범위: `<!-- shared-ai-context:start -->`부터 `<!-- shared-ai-context:end -->`까지 관리 블록 1회
- 백업:
  - 전역 `AGENTS.md` 생성 전 부재 상태와 초기 상태 파일: `C:\Users\jihon\.codex\backups\codex-v2-migration-20260727-203935-KST\`
  - Cursor 배포 커밋 후 `SYNC-STATUS.md` 재백업: `C:\Users\jihon\.codex\backups\codex-v2-migration-20260727-204435-KST\`
- 기존 `C:\Users\jihon\.codex\config.toml`: 읽기·점검만 했으며 공통 정책이나 프로젝트 사실을 추가하지 않았다.
- 검사 결과: 관리 블록 시작·끝 표식 각각 1개, `Managed policy version: 2` 1개, 기준 정책 경로·상대 Markdown 링크·비밀정보 서명 검사 통과.
- 정책 흐름: Vault 기준 문서는 수정하지 않고, 기준 → 배포본 방향으로만 전역 관리 블록을 반영했다.
- Inbox 점검: 문서 5개, `review_by` 도래·경과 문서 0개. 5개 누적 트리거가 충족되어 사용자 검토가 필요하며 자동 승격·삭제는 하지 않았다.
- 수동 보류: Vault 전체 검사에서 `20-projects/game-factory/PROJECT-CARD.md`의 기존 `\|` 포함 위키 링크 10개가 발견됐다. 대상 사고 파일은 존재하지만 링크 표기 의도는 확인이 필요하며, Codex 배포 범위와 무관해 수정하지 않았다.

기준과 실제 설정의 버전이 다르면 [[POLICY-DISTRIBUTION]]에 따라 기준에서 다시 배포한다.
