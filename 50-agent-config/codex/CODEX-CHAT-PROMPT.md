# Codex 채팅창용 마이그레이션 프롬프트

첨부한 `CODEX-MIGRATION-GUIDE.md`를 전부 읽고, 그 문서를 기준으로 이 컴퓨터의 Codex 전역 설정과 현재 열려 있는 프로젝트를 마이그레이션해줘.

작업 요구사항:

1. `C:\Users\jihon\.codex\AGENTS.md`, `config.toml`, 현재 프로젝트의 모든 적용 가능한 `AGENTS.md`, `.codex/config.toml`, `CLAUDE.md`, `GEMINI.md`, Cursor Rules를 먼저 읽어.
2. 기존 파일을 수정하기 전에 타임스탬프 백업을 만들어.
3. 전역 `AGENTS.md`에는 사용자 선호와 공통 컨텍스트 프로토콜만 병합하고 프로젝트별 사실을 넣지 마.
4. 현재 프로젝트의 `AGENTS.md`를 공통 기준으로 정리하되 기존 규칙과 사용자 변경을 보존해.
5. 기존 문서와 중복되지 않는 방식으로 `docs/ai/PROJECT.md`, `NOW.md`, `BACKLOG.md`, `decisions/` 구조를 도입해.
6. `NOW.md`는 실제 Git과 코드 상태를 조사해 작성하고 모르는 내용은 추측하지 마.
7. 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`의 작성 정책·프로필·Git 정책과 프로젝트 카드를 읽고 연결해.
8. 검증되지 않은 기록을 영구 사건이나 패턴으로 승격하지 마.
9. Push, Pull, Merge, Rebase, 브랜치 삭제, force push, hard reset은 실행하지 마.
10. 변경 후 Git diff, 문서 링크, 중복 규칙, 비밀정보 여부를 검증해.
11. 완료 시 백업 위치, 변경 파일, 병합 내용, 보류 사항, Git 상태, 검증 결과를 표로 보고해.

프로젝트 루트나 기존 규칙의 의도를 확정할 수 없으면 임의로 재구성하지 말고, 안전하게 확인된 범위까지만 수행한 뒤 질문이 필요한 항목을 보고해.
