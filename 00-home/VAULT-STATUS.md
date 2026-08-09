---
type: vault-status
schema_version: 2
policy_version: 2
updated: 2026-08-09
---

# Vault 현재 상태

## 구조 상태

- 스키마 버전: 2
- 공통 정책 버전: 2
- 마이그레이션 완료일: 2026-07-27
- 원본 백업: `AI-CONTEXT-LOGGING-pre-migration-20260727.zip`
- 검수 보고서: [[audits/2026-07-27-ai-context-migration-session-report]]
- v2 검수 보고서: [[audits/2026-07-27-ai-context-migration-v2-report]]
- Git 상태: `main`, v1 기준선 `877862c`
- 원격 저장소: `StockHedge/obsidian-ai-context` (GitHub Private, 2026-08-09 연결)
- 기기 간 동기화: [[VAULT-SYNC]] — PC(`pc-main`) ↔ 노트북(`laptop`), Obsidian Git 플러그인
- Obsidian 커뮤니티 플러그인: `Git` (Vinzent03) — 두 기기에 설치 필요

## 등록 프로젝트

| 프로젝트 | 상태 | 카드 |
|---|---|---|
| Game Factory | 진행중, 사건 로그 이관 완료 | [[20-projects/game-factory/PROJECT-CARD]] |
| Embolos | 진행중, 프로젝트 카드·Ops 마일스톤 이관 완료 | [[20-projects/embolos/PROJECT-CARD]] |

## Inbox

- Cursor: 컨텍스트 로깅 경로 정정 1건. Embolos 임시 인계·포인터 2건은 프로젝트 저장소로
  이관 완료 후 2026-08-09 커밋 `eb065ec`에서 정리 (이력에서 복구 가능)
- Claude Code, Codex, ChatGPT, Gemini, 기타: 현재 빈 수신함

## 운영상 남은 작업

- 각 실제 프로젝트에 `AGENTS.md`와 `docs/ai/` 구조 적용
- ~~`10-inbox/cursor`의 3개 임시 기록 처리~~ → 2026-08-09 완료 (2건 정리, 1건 유지)
- Claude·Codex·Cursor 실제 전역 설정에 v2 관리 블록을 적용하고 [[50-agent-config/SYNC-STATUS]] 갱신
- ~~비공개 원격 저장소 연결 여부 결정~~ → 2026-08-09 완료, [[VAULT-SYNC]] 참조
- ~~`Documents\Obsidian Vault`와 노트북 Vault를 이 Vault로 통합~~ → 2026-08-09 완료,
  [[audits/2026-08-09-vault-consolidation]] 참조
- 구 BugLog 11건의 프론트매터를 [[INCIDENT.template]] 스키마로 정규화
- 한글 파일·폴더명의 영문 슬러그화 — 링크 재작성과 반드시 함께 수행 (아래 「알려진 제약」)
- 구 `Documents\Obsidian Vault` 폴더를 Obsidian 등록 해제 후 백업·정리

## 알려진 제약

- 대시보드 자동 집계 플러그인에 의존하지 않도록 현재는 Markdown 링크 중심으로 구성했다.
- 통합된 카페24 레퍼런스·AIWebbuilder 영역은 [[WRITING-POLICY]]의 영문 파일명 규칙에서
  벗어나 있다. 첨부 220건과 AIWebbuilder MOC가 전부 `[[파일명]]` 최단 형식 링크를 쓰므로
  이름을 바꾸면 링크가 깨진다. 이름 변경은 링크 재작성과 한 작업으로 묶어야 한다.
- `20-projects/aiwebbuilder/`는 `incidents/·milestones/·retrospectives/` 대신 주제별 MOC
  구조를 쓴다. [[20-projects/README]] 규약에 대한 의도된 예외다.
- ChatGPT 웹은 이 로컬 Vault를 직접 읽을 수 없다. 파일 첨부나 별도 연결 수단이 필요하다.
- 로컬 파일 변경도 이미 진행 중인 모델 컨텍스트에 자동 주입되지 않는다.
- 프로젝트 코드와 Vault 기록이 충돌하면 코드, Git, 실제 검증 결과를 우선한다.
