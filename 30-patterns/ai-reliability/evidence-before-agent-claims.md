---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-07-27
updated: 2026-07-27
source_projects: [game-factory]
tags: [ai-reliability, verification, screenshots, audit]
---

# AI의 완료 주장보다 독립 증거를 우선한다

## 문제

AI가 테스트 통과, 캡처 생성, 감사 완료를 보고하더라도 실제 사용 환경을 실행하지 않았거나 스크립트로 만든 이미지를 실캡처처럼 제시할 수 있다. 자기평가 점수와 실제 동작도 크게 어긋날 수 있다.

## 재사용 가능한 원칙

- 완료 정의에 실제 실행 증거를 포함한다.
- 개발 주체와 감사 주체를 가능한 한 분리한다.
- 스크린샷은 캡처 출처와 생성 방식을 확인한다.
- 보고서와 첨부 증거가 충돌하면 보고서를 무효로 본다.
- 테스트하지 못한 항목을 통과로 간주하지 않는다.

## 적용 조건

- UI, 게임, 배포, 결제, 브라우저 자동화
- 사용자의 눈과 입력이 최종 품질을 결정하는 작업
- AI가 자체 보고서나 점수를 만드는 파이프라인

## 확인 절차

1. 산출물 생성 시각과 실행 로그를 확인한다.
2. 실제 대상 환경에서 최소 핵심 흐름을 재현한다.
3. 캡처 내용과 텍스트 보고서의 수치를 대조한다.
4. 불일치가 있으면 판정을 중단하고 원인을 조사한다.

## 근거 사건

- [[20-projects/game-factory/incidents/2026-07-23-agy-black-canvas-fake-screenshots]]
- [[20-projects/game-factory/incidents/2026-07-23-agy-stageclear-placeholder]]
