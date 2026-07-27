---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-07-27
updated: 2026-07-27
source_projects: [game-factory]
tags: [windows, path, encoding, android]
---

# Windows 도구 경로는 ASCII를 우선한다

## 문제

Android Emulator, Node 기반 도구, 일부 오래된 CLI는 한글 사용자명이나 비 ASCII 경로를 완전히 처리하지 못할 수 있다. 증상은 파일 미발견, 부팅 실패, 파라미터 로드 실패처럼 나타난다.

## 재사용 가능한 원칙

- SDK, 캐시, 빌드, 임시 실행 경로는 가능한 한 짧은 ASCII 경로를 사용한다.
- 기존 설치를 옮기기 어렵다면 검증된 junction을 사용한다.
- 경로 오류가 의심되면 코드 수정 전에 ASCII 경로에서 같은 명령을 재현한다.

## 확인 절차

1. 오류에 표시된 모든 절대 경로를 확인한다.
2. 사용자명과 상위 폴더에 한글·공백·특수문자가 있는지 확인한다.
3. ASCII 경로로 우회해 같은 동작을 비교한다.
4. 성공하면 우회 경로와 환경변수를 프로젝트 문서에 고정한다.

## 근거 사건

- [[20-projects/game-factory/incidents/env-emulator-korean-path-panic]]
