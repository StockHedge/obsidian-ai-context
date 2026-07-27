---
type: vault-status
schema_version: 1
updated: 2026-07-27
---

# Vault 현재 상태

## 구조 상태

- 스키마 버전: 1
- 마이그레이션 완료일: 2026-07-27
- 원본 백업: `AI-CONTEXT-LOGGING-pre-migration-20260727.zip`
- 검수 보고서: [[audits/2026-07-27-ai-context-migration-session-report]]
- Git 상태: 아직 Git 저장소가 아님
- Obsidian 커뮤니티 플러그인: 확인된 설치 없음

## 등록 프로젝트

| 프로젝트 | 상태 | 카드 |
|---|---|---|
| Game Factory | 진행중, 사건 로그 이관 완료 | [[20-projects/game-factory/PROJECT-CARD]] |
| Embolos | 진행중, 프로젝트 카드·Ops 마일스톤 이관 완료 | [[20-projects/embolos/PROJECT-CARD]] |

## Inbox

- Cursor: 컨텍스트 로깅 경로 정정 1건, Embolos 임시 인계와 포인터 2건
- Claude Code, Codex, ChatGPT, Gemini, 기타: 현재 빈 수신함

## 운영상 남은 선택

- Vault를 비공개 Git 저장소 또는 다른 버전 백업에 연결할지 결정
- 각 실제 프로젝트에 `AGENTS.md`와 `docs/ai/` 구조 적용
- `10-inbox/cursor`의 3개 임시 기록을 7일 내 승격·프로젝트 이관·보관 판단
- 수동 프로젝트 카드의 주기적 검토 담당 방식 결정

## 알려진 제약

- 대시보드 자동 집계 플러그인에 의존하지 않도록 현재는 Markdown 링크 중심으로 구성했다.
- ChatGPT 웹은 이 로컬 Vault를 직접 읽을 수 없다. 파일 첨부나 별도 연결 수단이 필요하다.
- 프로젝트 코드와 Vault 기록이 충돌하면 코드, Git, 실제 검증 결과를 우선한다.
