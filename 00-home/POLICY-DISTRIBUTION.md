---
type: operating-policy
schema_version: 2
policy_version: 2
updated: 2026-07-27
---

# 공통 정책 배포 규칙

## 단방향 원칙

공통 정책은 반드시 다음 방향으로만 흐른다.

`Vault 기준 문서 → 50-agent-config 배포 템플릿 → 각 AI의 실제 전역 설정`

각 AI의 전역 설정에 배포된 관리 블록은 복사본이다. 그 블록을 직접 고쳐 기준 문서로 역수입하지 않는다.

## 기준 문서

| 정책 | 기준 위치 |
|---|---|
| Vault 기록·승격 | [[WRITING-POLICY]] |
| 사용자에 대한 확인된 사실 | [[USER-PROFILE]] |
| 공통 작업 선호 | [[WORK-PREFERENCES]] |
| Git 행동 권한 | [[GIT-POLICY]] |
| 동기화 가능 범위 | [[SYNC-BOUNDARIES]] |
| Inbox 검토 | [[10-inbox/INBOX-REVIEW]] |

## 배포 절차

1. 기준 문서를 먼저 수정한다.
2. `policy_version`을 올리고 변경 이유를 기록한다.
3. `50-agent-config/`의 도구별 가이드와 관리 블록을 갱신한다.
4. 각 AI의 기존 전역 설정을 백업하고 관리 블록만 병합한다.
5. 실제 설정에서 중복·충돌·경로 오류를 확인한다.
6. [[50-agent-config/SYNC-STATUS]]에 배포 결과와 날짜를 기록한다.

## 드리프트 판정

- 실제 전역 설정의 `policy_version`이 기준보다 낮으면 재배포 대상이다.
- 실제 전역 설정에만 존재하는 공통 정책은 기준 문서에 검토 제안으로 올린 뒤 채택 여부를 결정한다.
- 프로젝트별 규칙은 전역 설정이나 `40-profile/`로 승격하지 않는다.
- 자동으로 기존 사용자 규칙을 삭제하거나 전체 파일을 덮어쓰지 않는다.

