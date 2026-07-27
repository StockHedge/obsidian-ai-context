---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-07-27
updated: 2026-07-27
source_projects: [game-factory]
tags: [runtime-parity, webview, mobile, verification]
---

# 최종 대상 런타임에서 최소 스모크 테스트를 수행한다

## 문제

데스크톱 브라우저 테스트가 통과해도 Android WebView, 실제 기기, 운영 네트워크처럼 최종 런타임이 다르면 부팅·입력·내비게이션 결함이 남을 수 있다.

## 재사용 가능한 원칙

- 최종 배포 환경과 다른 테스트만으로 완료를 선언하지 않는다.
- 비싼 전체 E2E 전에 설치·부팅·첫 입력·핵심 화면 전환을 확인하는 짧은 스모크 게이트를 둔다.
- 런타임별 콘솔과 시스템 로그를 수집할 경로를 미리 마련한다.

## 확인 절차

1. 배포 산출물이 최신 코드에서 만들어졌는지 확인한다.
2. 실제 대상 또는 가장 가까운 에뮬레이터에 설치한다.
3. 앱 부팅, 첫 입력, 핵심 전환 한 번을 검증한다.
4. 실패 시 대상 런타임 로그로 최초 오류를 찾는다.

## 근거 사건

- [[20-projects/game-factory/incidents/2026-07-24-crowdclash-webview-defects]]
