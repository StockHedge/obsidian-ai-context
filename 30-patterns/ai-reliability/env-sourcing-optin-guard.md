---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-08-03
updated: 2026-08-03
source_projects: [embolos]
agents: [claude-code]
tags: [env, shell, pytest, prod-safety, structural-guard]
---

# 위험 환경은 opt-in 마커로만 열린다 (env-sourcing guard)

## 문제

로컬 기본 환경(.env)이 prod 같은 위험 대상을 가리킬 때, "테스트 전에 안전 env를
소싱하라"는 규율은 셸 체인 한 조각의 실패로 조용히 무너진다. 실사고:
`lint && SP=…; . "$SP/safe.env"; pytest`에서 lint 실패 → `&&` 단락 → SP 빈 문자열 →
소싱 실패(무시됨) → pytest가 prod로 실행 (embolos 2026-08-01,
[[2026-08-01-pytest-prod-db-near-miss]]).

## 재사용 가능한 원칙

1. **안전한 환경이 자신을 증명해야 실행이 열린다.** 안전 env 파일에만 존재하는
   마커 변수(`PYTEST_DB_OK=1` 등)를 정의하고, 테스트 하네스 진입점(conftest 최상단
   등)에서 마커 부재 시 즉시 하드 중단한다. 검사 방향이 "위험을 감지"(블록리스트)가
   아니라 "안전을 증명"(opt-in)이어야 미지의 실패 경로에서도 닫힌다.
2. env 소싱은 명령 체인 **맨 앞** + `|| exit`. 린트·빌드 등 실패 가능 명령과
   `&&`로 묶지 않는다.
3. 시크릿 마스킹 정규식은 접미 변형까지: `(PASSWORD|SECRET|KEY|URL[A-Z_]*)=`.

## 적용 조건

- 로컬 기본 설정이 위험 대상(prod DB·실결제·실발신)을 가리키는 모든 프로젝트.
- AI 에이전트가 셸 명령을 조립해 실행하는 워크플로(체인 실패가 조용히 전파됨).

## 적용하지 말아야 할 조건

- 기본 환경 자체가 이미 안전(로컬 컨테이너 DB 등)이라 마커가 의식 없는 통과가
  되는 경우 — 가드가 형식화되면 신호 가치가 죽는다.

## 확인 절차

마커 없이 하네스를 실행해 **즉시 중단되는지**를 한 번 실측한다(가드의 음성 대조군).

## 근거 사건

[[2026-08-01-pytest-prod-db-near-miss]] — 오염 0건으로 끝났지만, prod 스키마가
같았다면 침묵 진행했을 니어미스.
