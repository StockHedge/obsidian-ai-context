---
type: vault-status
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# Vault 현재 상태

## 구조 상태

- 스키마 버전: 2
- 공통 정책 버전: 2
- 마이그레이션 완료일: 2026-07-27
- 원본 백업: `AI-CONTEXT-LOGGING-pre-migration-20260727.zip`
- 검수 보고서: [[audits/2026-07-27-ai-context-migration-session-report]]
- v2 검수 보고서: [[audits/2026-07-27-ai-context-migration-v2-report]]
- Git 상태: 로컬 저장소, `main`, v1 기준선 `877862c`
- 원격 저장소: 연결하지 않음
- Obsidian 커뮤니티 플러그인: 확인된 설치 없음

## 등록 프로젝트

| 프로젝트 | 상태 | 카드 |
|---|---|---|
| Game Factory | 진행중, 사건 로그 이관 완료 | [[20-projects/game-factory/PROJECT-CARD]] |
| Embolos | 진행중, 프로젝트 카드·Ops 마일스톤 이관 완료 | [[20-projects/embolos/PROJECT-CARD]] |

## Inbox

- Cursor: 컨텍스트 로깅 경로 정정 1건, Embolos 임시 인계와 포인터 2건
- Claude Code, Codex, ChatGPT, Gemini, 기타: 현재 빈 수신함

## 운영상 남은 작업

- 각 실제 프로젝트에 `AGENTS.md`와 `docs/ai/` 구조 적용
- `10-inbox/cursor`의 3개 임시 기록을 `review_by`까지 승격·프로젝트 이관·보관 판단
- Claude·Codex·Cursor 실제 전역 설정에 v2 관리 블록을 적용하고 [[50-agent-config/SYNC-STATUS]] 갱신
- 필요하면 비공개 원격 저장소 연결 여부를 별도로 결정

## 알려진 제약

- 대시보드 자동 집계 플러그인에 의존하지 않도록 현재는 Markdown 링크 중심으로 구성했다.
- ChatGPT 웹은 이 로컬 Vault를 직접 읽을 수 없다. 파일 첨부나 별도 연결 수단이 필요하다.
- 로컬 파일 변경도 이미 진행 중인 모델 컨텍스트에 자동 주입되지 않는다.
- 프로젝트 코드와 Vault 기록이 충돌하면 코드, Git, 실제 검증 결과를 우선한다.
