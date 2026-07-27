---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-07-27
updated: 2026-07-27
source_projects: [game-factory]
tags: [localhost, ports, cache, service-worker]
---

# 로컬 웹 프로젝트는 origin을 분리한다

## 문제

서로 다른 앱이 같은 localhost 포트를 반복해서 사용하면 브라우저 캐시, 서비스 워커, IndexedDB, localStorage가 같은 origin에 묶여 이전 앱의 자산과 상태가 섞일 수 있다.

## 재사용 가능한 원칙

- 동시에 다루는 앱마다 고유한 개발 포트를 고정한다.
- 포트는 프로젝트 문서와 실행 스크립트에 명시한다.
- 이상한 교차 로딩이 보이면 코드보다 캐시·서비스 워커·origin 재사용을 먼저 확인한다.

## 확인 절차

1. 현재 포트를 사용하는 프로세스를 확인한다.
2. 브라우저 Application 영역에서 서비스 워커와 저장소를 확인한다.
3. 새 포트로 실행해 증상을 비교한다.
4. 필요하면 기존 origin 저장소를 명시적으로 제거한다.

## 근거 사건

- [[20-projects/game-factory/incidents/env-local-port-collision]]
