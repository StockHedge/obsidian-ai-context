---
document_type: migration-chat-prompt
target: codex
version: 2
policy_version: 2
---

# Codex 채팅창용 v2 마이그레이션 프롬프트

첨부한 `CODEX-MIGRATION-GUIDE.md` v2를 전부 읽고, 그 문서를 최우선 절차로 삼아 이 컴퓨터의 Codex 전역 설정과 현재 프로젝트를 마이그레이션해줘.

먼저 중앙 Vault `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`가 Git 저장소인지, 현재 변경이 다른 작성자와 안전하게 분리되는지 확인해. 이상이 있으면 쓰기를 멈추고 원인을 보고해.

요구사항:

1. 전역·프로젝트 `AGENTS.md`, `config.toml`, 다른 AI 규칙, 프로젝트 Git 상태를 먼저 읽어.
2. 수정 전 타임스탬프 백업을 만들고 사용자 변경을 보존해.
3. 전역 관리 블록은 `policy_version 2`로 한 번만 병합하고 프로젝트 사실을 전역 설정이나 메모리에 넣지 마.
4. 기존 문서와 중복되지 않게 `AGENTS.md`와 `docs/ai/` 기준 구조를 적용해.
5. `NOW.md`는 실제 코드·Git·검증 결과로 작성하고 추측은 `확인 필요`로 표시해.
6. Inbox 검토 트리거와 정책 단방향 배포 규칙을 적용해.
7. Push, Pull, Merge, Rebase, 브랜치·태그 삭제, 원격 생성·변경은 실행하지 마.
8. 변경 후 diff, 지침 중복, 링크, 경로, 비밀정보를 검증해.
9. 완료 후 `50-agent-config/SYNC-STATUS.md`를 실제 결과로 갱신하고 백업·변경 파일·Git 상태·검증·보류 항목을 보고해.

프로젝트 루트나 기존 규칙의 의도를 확정할 수 없으면 임의로 재구성하지 말고 확인된 범위까지만 진행해.
