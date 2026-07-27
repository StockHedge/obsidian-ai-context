---
type: agent-config-sync-status
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# AI 전역 설정 배포 상태

| 대상 | 기준 정책 | 템플릿 | 실제 설정 반영 | 마지막 확인 |
|---|---:|---:|---|---|
| Claude Code | 2 | 2 | 미확인 | 2026-07-27 |
| Codex | 2 | 2 | 미확인 | 2026-07-27 |
| Cursor User Rules | 2 | 2 | 미확인 | 2026-07-27 |

`미확인`은 적용 실패가 아니라, 이 Vault v2 작업에서 실제 전역 설정을 수정하지 않았다는 뜻이다.

각 도구에서 마이그레이션을 실행한 뒤 다음을 기록한다.

- 적용한 `policy_version`
- 실제 설정 파일 또는 UI 위치
- 백업 위치
- 적용 시각
- 중복·충돌 검사 결과
- 수동 보류 항목

기준과 실제 설정의 버전이 다르면 [[POLICY-DISTRIBUTION]]에 따라 기준에서 다시 배포한다.
