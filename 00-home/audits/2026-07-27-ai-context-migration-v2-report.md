---
type: audit-report
schema_version: 2
policy_version: 2
status: completed
created: 2026-07-27
agents: [codex]
tags: [ai-context, migration, audit, v2]
---

# AI Context Logging v2 종합 검수 보고서

## 1. 검수 대상과 요청

이번 v2 작업은 다음 요청을 하나의 범위로 처리했다.

1. 중앙 Obsidian Vault를 여러 AI가 함께 사용할 수 있는 구조로 재설계
2. 기존 기록을 보존하며 프로젝트·공통 지식 중심으로 이관
3. Claude Code, Codex, Cursor용 마이그레이션 가이드와 채팅 프롬프트 작성
4. AI 자동 기록 외에 사용자가 직접 갱신할 시점 정의
5. v1 검수에 대한 외부 AI의 지적을 실제 파일과 운영 규칙에 반영
6. 수정된 전체 문서를 v2 패키지로 다시 제공

대상 Vault:

`C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`

## 2. 최종 판정

v2 구조·정책·배포 문서 작성은 완료됐다. v1의 설계는 유지하면서 버전 관리, Inbox 집행 트리거, 정책 드리프트 방지, 동기화 한계, 프로젝트 카드 형식, 비공유 메모리 참조를 보강했다.

단, 이번 작업은 Claude Code·Codex·Cursor의 실제 전역 설정이나 각 프로젝트 저장소까지 자동 배포한 작업은 아니다. 실제 배포 상태는 [[50-agent-config/SYNC-STATUS]]에서 `미확인`으로 명시했다.

## 3. v2의 기준 구조

| 계층 | 기준 정보 | 역할 |
|---|---|---|
| 프로젝트 저장소 | 코드, Git, `AGENTS.md`, `docs/ai/` | 현재 상태와 실행 규칙 |
| Vault `20-projects/` | 검증된 사건·마일스톤·회고 | 프로젝트 장기 기록 |
| Vault `30-patterns/` | 일반화된 교훈 | 프로젝트 간 재사용 |
| Vault `40-profile/` | 사용자 성향·공통 운영 정책 | 공통 기준 |
| Vault `50-agent-config/` | 도구별 배포 템플릿 | 실제 설정의 복사 원본 |
| AI 내장 메모리 | 보조 회상 | 기준 정보 아님 |

공통 정책은 `Vault 기준 문서 → 50-agent-config → 실제 전역 설정`으로만 배포한다.

## 4. v1 검토 의견의 위험-조치-검증

| 위험 | v2 조치 | 검증 결과 | 판정 |
|---|---|---|---|
| 다중 작성자 Vault에 버전 관리가 없음 | 로컬 Git 저장소와 `main` 생성, `.gitignore`, `.gitattributes`, v1 기준선 커밋 | 기준선 `877862c`, `.obsidian/workspace.json` 무시 확인 | 해소 |
| Game Factory 날짜·상태 해소가 보고서에 없음 | 카드에 `updated`, `last_verified`, 검토일과 사건별 상태 유지 | 2026-07-27 기준 9건 해결, 1건 진행중 표기 확인 | 해소 |
| Claude 로컬 메모리 경로·식별자 종속 | 사건 6건을 공용 패턴 링크와 역사 식별자로 정규화 | Claude 전용 경로 문자열과 과거 메모리 라벨 0건 | 해소 |
| 웹 AI까지 실시간 자동 동기화된다는 오해 | [[SYNC-BOUNDARIES]]에 로컬·웹·대화 컨텍스트 한계 명시 | 가이드 3종 관리 블록에도 재로딩 원칙 포함 | 제약 명시 |
| Inbox를 비우는 주기·담당이 없음 | [[10-inbox/INBOX-REVIEW]]에 주간, 기한 도래, 5건 누적, 단계 종료 트리거와 사용자 최종 소유권 정의 | 기존 Cursor 초안 3건에 `review_by: 2026-08-03` 확인 | 운영 규칙 해소 |
| Vault 정책과 AI 전역 설정의 드리프트 | [[POLICY-DISTRIBUTION]]과 `policy_version: 2`, [[50-agent-config/SYNC-STATUS]] 도입 | 3개 도구 가이드·프롬프트가 버전 2로 일치 | 해소 |
| Embolos 카드가 Project Card 형식을 따르지 않음 | 실제 저장소를 읽고 v2 템플릿의 목적·기준 위치·구조·제약·단계·검증 형식으로 재작성 | 필수 9개 섹션과 저장소 상태 확인 | 해소 |
| `30-patterns` 파일명 ASCII 원칙 확인 필요 | 전체 하위 파일명 검사 | 비 ASCII 파일명 0건 | 해소 |

## 5. 버전 관리 조치

- 로컬 Git 저장소 초기화
- 기본 브랜치: `main`
- v1 기준선 커밋: `877862c chore: establish v1 context vault baseline`
- 원격 저장소: 생성·연결하지 않음
- Push·Pull: 실행하지 않음
- Obsidian의 변동성 높은 `workspace*.json`, 캐시, 임시 파일을 추적 제외
- v1 문서는 기준선 커밋에서 복원·비교 가능

ZIP 백업은 계속 보조 복구 수단으로 보존하지만 이후 변경 비교의 기준은 Git이다.

## 6. 새 운영 문서

- [[VERSION-CONTROL]]: Vault Git 시작 전 점검, 커밋, 복구
- [[POLICY-DISTRIBUTION]]: 기준 정책의 단방향 배포와 드리프트 판정
- [[SYNC-BOUNDARIES]]: 로컬 파일 공유와 모델 컨텍스트의 차이
- [[USER-MAINTENANCE]]: 사용자가 직접 확인·결정해야 하는 시점
- [[10-inbox/INBOX-REVIEW]]: Inbox 검토 트리거, 담당, 처리 선택
- [[50-agent-config/SYNC-STATUS]]: 실제 전역 설정의 배포 상태

## 7. 프로젝트 기록 수정

### Game Factory

- 카드 스키마를 v2로 올리고 마지막 검증일을 명시했다.
- 과거 Claude 전용 메모리 참조를 제거했다.
- 사건 6건은 원문 식별자를 역사 기록으로 남기되 공용 `30-patterns` 링크를 기준으로 연결했다.
- 현재 사건 표는 2026-07-27 기준 9건 해결, 1건 진행중이다.

저장소 경로 자체는 이번 작업에서 확인하지 못했으므로 카드의 `repo_path`는 비워 두고 추측하지 않았다.

### Embolos

실제 저장소 `C:\Users\jihon\projects\embolos`를 읽어 카드를 다시 작성했다.

- 브랜치: `legal-kakaopay-2026-07`
- 확인한 HEAD: `459a5c0`
- 확인된 최신 범위: AI Company Ops 콘솔과 제어면 관련 커밋
- 작업 트리 주의: `docs.zip`, `docs/handoff/` 미추적
- 프로젝트 `AGENTS.md`, `docs/ai/NOW.md`: 확인 시 없음

미추적 파일은 수정·커밋·삭제하지 않았다. 운영 배포, 결제, OAuth 같은 외부 상태는 문서만으로 완료 처리하지 않고 재검증 대상으로 표시했다.

## 8. AI별 v2 마이그레이션 세트

Claude Code, Codex, Cursor 각각에 다음 두 문서를 새로 작성했다.

- 도구별 설정 위치와 제약을 반영한 마이그레이션 가이드
- 해당 가이드를 첨부한 뒤 채팅창에 붙여넣는 실행 프롬프트

모든 v2 세트에는 다음이 공통으로 들어간다.

- Vault Git 점검을 0단계로 수행
- 기존 설정 백업과 병합
- 관리 블록 `policy_version 2`
- 공통 정책의 단방향 배포
- Inbox 검토 트리거
- 로컬 파일과 진행 중 대화의 비실시간성
- 실제 코드·Git·테스트 증거 우선
- 원격·병합·삭제 작업 제한
- 배포 후 `SYNC-STATUS` 갱신

## 9. 검증 결과

| 검사 | 결과 |
|---|---|
| Markdown UTF-8 엄격 디코딩 | 오류 0건 |
| frontmatter 종료 구분자 | 오류 0건 |
| 고신뢰 비밀키 패턴 | 0건 |
| Claude 로컬 메모리 경로·라벨 | 0건 |
| `30-patterns` 비 ASCII 파일명 | 0건 |
| Embolos Project Card 필수 섹션 | 통과 |
| v2 가이드·프롬프트 버전 일치 | 통과 |
| Obsidian workspace 파일 Git 무시 | 통과 |
| Wiki 링크 | v2 보고서 생성 후 재검사 통과 |

## 10. 하지 않은 일

- Claude Code, Codex, Cursor의 실제 전역 설정 수정
- Embolos 또는 Game Factory 저장소에 `AGENTS.md`, `docs/ai/` 생성
- Vault 원격 저장소 생성·연결·Push
- ChatGPT·Gemini 웹에 로컬 Vault 커넥터 연결
- Inbox 자동 삭제·자동 승격
- Embolos의 미추적 파일 변경
- 외부 서비스의 운영 상태 확인

## 11. 남은 사용자 작업

1. 새 v2 가이드와 프롬프트를 Claude Code, Codex, Cursor에 각각 적용한다.
2. 적용 직후 [[50-agent-config/SYNC-STATUS]]를 실제 결과로 갱신한다.
3. 2026-08-03까지 Cursor Inbox 3건을 검토한다.
4. 각 프로젝트에 `AGENTS.md`와 `docs/ai/NOW.md`를 도입할 때 실제 저장소 증거로 작성한다.
5. 필요할 때만 Vault의 비공개 원격 저장소 연결 여부를 결정한다.

## 12. 최종 결론

v2는 “모든 AI의 메모리가 실시간으로 합쳐지는 시스템”이 아니다. 대신 코드·Git·프로젝트 인계·검증된 지식의 소유 위치를 고정하고, 여러 AI가 같은 근거를 다시 읽을 수 있게 만든 운영 체계다.

가장 중요한 변화는 로컬 Git 기준선, Inbox 집행 트리거, 정책 단방향 배포다. 이 세 가지가 v1에서 남았던 운영 리스크를 실제 절차로 닫는다.
