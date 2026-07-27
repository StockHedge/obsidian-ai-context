---
document_type: migration-chat-prompt
target: cursor
version: 2
policy_version: 2
---

# Cursor 채팅창용 v2 마이그레이션 프롬프트

첨부한 `CURSOR-MIGRATION-GUIDE.md` v2를 전부 읽고, 그 문서를 최우선 절차로 삼아 Cursor 설정과 현재 프로젝트를 마이그레이션해줘.

먼저 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`가 Git 저장소인지, 다른 Cursor 세션이나 AI의 변경과 안전하게 분리되는지 확인해. 이상이 있으면 같은 파일 쓰기를 멈추고 원인을 보고해.

요구사항:

1. User Rules, `.cursor/rules`, `.cursorrules`, `.cursor/commands`, `AGENTS.md`, 다른 AI 규칙과 프로젝트 Git 상태를 먼저 읽어.
2. 프로젝트 파일 수정 전 타임스탬프 백업을 만들고 기존 규칙을 보존해.
3. User Rules 관리 블록은 `policy_version 2`로 한 번만 병합해. 직접 수정할 수 없으면 적용했다고 주장하지 말고 정확한 붙여넣기 블록을 출력해.
4. `AGENTS.md`를 도구 중립적 기준으로 두고 `.cursor/rules/shared-context.mdc`를 얇은 어댑터로 구성해.
5. 기존 문서와 중복되지 않게 `docs/ai/` 기준 구조를 적용해.
6. `NOW.md`는 실제 코드·Git·검증 결과로 작성하고 추측은 `확인 필요`로 표시해.
7. Inbox 검토 트리거와 정책 단방향 배포 규칙을 적용해.
8. `.cursorrules`를 검토 없이 삭제하지 마.
9. Push, Pull, Merge, Rebase, 브랜치·태그 삭제, 원격 생성·변경은 실행하지 마.
10. 완료 후 `50-agent-config/SYNC-STATUS.md`를 실제 결과로 갱신하고 백업·변경 파일·User Rules 상태·Git 상태·검증·보류 항목을 보고해.

UI나 권한 제약으로 못 한 일은 우회했다고 가장하지 말고 사용자가 해야 할 정확한 단계로 남겨줘.
