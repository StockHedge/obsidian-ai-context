---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-07-27
updated: 2026-07-27
source_projects: [game-factory]
tags: [debugging, permissions, deployment, diagnosis]
---

# 같은 증상이라도 원인은 세부 증거로 구분한다

## 문제

“제출 버튼이 비활성”, “커밋이 실패”, “403”처럼 비슷한 표면 증상은 권한, 정책 선언, 등재정보 검사 등 서로 다른 원인에서 발생할 수 있다.

## 재사용 가능한 원칙

- 증상 이름을 원인처럼 사용하지 않는다.
- 오류 코드, 상세 문제 보기, 계정 권한, 정책 게이트를 각각 확인한다.
- 403은 먼저 인증·인가 문제로 분류하되 실제 응답 본문으로 확인한다.
- 한 프로젝트에서 통과한 콘솔 설정이 다른 앱에도 적용됐다고 가정하지 않는다.

## 확인 절차

1. 가장 구체적인 오류 코드와 상세 메시지를 확보한다.
2. 계정·앱·환경별 설정 범위를 확인한다.
3. 원인 후보마다 독립적으로 반증 가능한 검사를 만든다.
4. 수정 후 같은 경로를 다시 실행해 원인을 확인한다.

## 근거 사건

- [[20-projects/game-factory/incidents/2026-07-22-adid-declaration-submit-block]]
- [[20-projects/game-factory/incidents/2026-07-22-edits-commit-403-permission]]
- [[20-projects/game-factory/incidents/2026-07-22-listing-prereview-false-positive]]
