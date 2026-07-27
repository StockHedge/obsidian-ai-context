# Cursor 채팅창용 마이그레이션 프롬프트

첨부한 `CURSOR-MIGRATION-GUIDE.md`를 전부 읽고 그 문서를 기준으로 현재 Cursor 설정과 열려 있는 프로젝트를 마이그레이션해줘.

반드시 다음을 지켜줘.

1. 기존 Cursor User Rules, `.cursor/rules`, `.cursorrules`, `.cursor/commands`, 루트 `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`를 먼저 조사해.
2. 프로젝트 파일 변경 전 타임스탬프 백업을 만들어.
3. 기존 규칙을 덮어쓰거나 중복시키지 말고 공통 규칙과 Cursor 전용 규칙을 분리해.
4. User Rules를 직접 안전하게 수정할 수 없다면 수정했다고 주장하지 말고, 가이드의 User Rules 블록을 내가 붙여넣을 수 있게 정확히 출력해.
5. 프로젝트 `AGENTS.md`를 도구 중립적 기준으로 정리하고 `.cursor/rules/shared-context.mdc`를 얇은 어댑터로 구성해.
6. 기존 문서와 중복되지 않게 `docs/ai/PROJECT.md`, `NOW.md`, `BACKLOG.md`, `decisions/` 구조를 도입해.
7. 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`의 작성 정책·프로필·Git 정책과 프로젝트 카드를 읽고 연결해.
8. `NOW.md`와 프로젝트 문서는 실제 코드·Git 상태를 조사해 작성하고 모르는 내용을 추측하지 마.
9. `.cursorrules`는 검토 없이 삭제하지 마.
10. Push, Pull, Merge, Rebase, 브랜치 삭제, force push, hard reset은 실행하지 마.
11. 완료 후 백업 위치, 변경 파일, User Rules 반영 상태, 레거시 규칙 처리, 보류 사항, 검증 결과를 표로 보고해.

권한이나 UI 제약으로 할 수 없는 작업은 다른 방식으로 처리했다고 가장하지 말고 사용자가 직접 해야 할 정확한 단계로 남겨줘.
