---
type: session-draft
schema_version: 2
status: draft
project: embolos
agents: [cursor]
created: 2026-07-27
review_by: 2026-08-03
contains_verified_result: true
tags: [migration, cursor, policy-v2]
---

# Cursor AI Context v2 적용 초안

## 검증된 결과

- User Rules에 `[Shared AI Context Protocol]` `Managed policy version: 2` 1회 반영 (id `16992538`).
- 중복 id `16992532` 제거.
- embolos에 `AGENTS.md`, `.cursor/rules/shared-context.mdc`, `docs/ai/*` 신설.
- `ops.test.embolos.kr/health` → HTTP 200.
- `test.embolos.kr/health` → 8s 타임아웃 (미해소).

## 승격 후보

- 공통 배포 결과는 이미 `50-agent-config/SYNC-STATUS.md`에 기록됨.
- 프로젝트 상시 인계는 repo `docs/ai/NOW.md`가 기준.
- 이 Inbox 초안은 주간 검토 때 보관/삭제 판단.

## 비고

기존 `2026-07-27-embolos-migration-handoff.md`는 제품 이관용이며, AI Context v2 적용만으로 자동 삭제하지 않음.
