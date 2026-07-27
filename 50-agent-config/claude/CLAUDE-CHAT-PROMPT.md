# Claude Code 채팅창용 마이그레이션 프롬프트

첨부한 `CLAUDE-MIGRATION-GUIDE.md`를 전부 읽고 그 문서를 기준으로 이 컴퓨터의 Claude Code 전역 설정과 현재 열려 있는 프로젝트를 마이그레이션해줘.

반드시 다음 순서를 지켜줘.

1. 전역 및 프로젝트의 기존 `CLAUDE.md`, `CLAUDE.local.md`, `.claude/rules`, `AGENTS.md`와 관련 설정을 먼저 읽어.
2. 변경 전에 타임스탬프가 포함된 백업을 만들어.
3. 기존 내용을 덮어쓰거나 중복 규칙을 추가하지 말고, 가이드의 관리 구간을 기존 규칙과 충돌 없이 병합해.
4. 전역에는 사용자 선호와 공통 프로토콜만 넣고 프로젝트별 사실은 넣지 마.
5. 현재 프로젝트에는 공통 기준인 `AGENTS.md`와 `docs/ai/PROJECT.md`, `NOW.md`, `BACKLOG.md`, `decisions/` 구조를 기존 문서와 중복되지 않게 도입해.
6. 루트 `CLAUDE.md`가 `@AGENTS.md`를 가져오도록 하되 기존 Claude 전용 지침은 보존해.
7. 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`의 `00-home/WRITING-POLICY.md`와 `40-profile/GIT-POLICY.md`를 읽고 프로젝트 카드를 연결해.
8. 실제 코드, Git 상태, 기존 문서를 바탕으로 내용을 작성하고 모르는 프로젝트 정보는 추측하지 말고 `확인 필요`로 표시해.
9. Push, Pull, Merge, Rebase, 브랜치 삭제는 실행하지 마.
10. 완료 후 백업 위치, 변경 파일, 병합한 규칙, 보류 사항, 검증 결과를 표로 보고해.

설정 충돌이나 권한 문제가 있으면 임의로 우회하거나 다른 위치에 복제하지 말고, 안전한 범위까지 진행한 뒤 정확한 차단 원인을 보고해.
