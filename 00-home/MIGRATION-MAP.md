---
type: migration-map
schema_version: 1
date: 2026-07-27
---

# 2026-07-27 Vault 마이그레이션 매핑

기존 파일은 이동 전에 ZIP으로 백업했다. 내용이 있는 문서는 삭제하지 않았다.

## 프로젝트 자료

| 이전 위치 | 새 위치 |
|---|---|
| `PROJECTS/GameFactory-BugLog/_dashboard.md` | `20-projects/game-factory/PROJECT-CARD.md` |
| `PROJECTS/GameFactory-BugLog/logs/` | `20-projects/game-factory/incidents/` |
| `PROJECTS/Embolos/남은 Phase와 트랙.md` | `20-projects/embolos/PROJECT-CARD.md` |
| `AI-CONTEXT-LOGGING/CursorAI/2026-07-26_Embolos_AI-Company-Ops-Console.md` | `20-projects/embolos/milestones/2026-07-26-ai-company-ops-console.md` |

## AI별 공간

| 이전 위치 | 새 위치 | 역할 변화 |
|---|---|---|
| `AI-CONTEXT-LOGGING/ChatGPT/` | `10-inbox/chatgpt/` | 임시 수신함 |
| `AI-CONTEXT-LOGGING/Claude/` | `10-inbox/claude-code/` | 임시 수신함 |
| `AI-CONTEXT-LOGGING/CursorAI/` | `10-inbox/cursor/` | 임시 수신함 |
| `AI-CONTEXT-LOGGING/Gemini/` | `10-inbox/gemini/` | 임시 수신함 |
| `AI-CONTEXT-LOGGING/TheOtherThings/` | `10-inbox/other/` | 임시 수신함 |
| 없음 | `10-inbox/codex/` | 새 임시 수신함 |

마이그레이션 도중 추가된 Cursor의 `2026-07-27_세션로그_컨텍스트로깅-경로정정.md`는 손실 없이 `10-inbox/cursor/`에 함께 보존했다.

구조화 이후 Cursor가 추가한 Embolos 임시 인계 문서와 포인터는 영구 프로젝트 지식과 구분하기 위해 `10-inbox/cursor/`로 정리했다.

## 제거된 빈 컨테이너

- `PROJECTS/`
- 중첩된 `AI-CONTEXT-LOGGING/`

두 폴더는 내용 이동 후 비어 있는 것을 확인한 다음 제거했다. 복원용 ZIP에는 이전 구조가 남아 있다.
