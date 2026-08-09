---
type: incident
schema_version: 1
project: for-marketing
component: backend/tests
category: test-isolation
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-03
resolved: 2026-08-03
verified: 2026-08-04
agents: [claude-code]
source_repo: C:\Users\강지호\project\for-marketing
source_commit: fd1a21a
tags: [pytest, secrets, side-effects]
---

# 테스트가 로컬 .env의 실자격증명을 상속해 실제 외부 API를 호출함

## 요약

P3 승인 파이프라인 테스트 실행 중, pydantic-settings가 `backend/.env`(실 Telegram 봇
토큰·Anthropic API 키 보유)를 로딩해 **테스트가 실제 Telegram 채팅으로 가짜 승인 카드를
발송**하고 실 Anthropic 호출이 발생했다.

## 증상

- 사용자 실제 Telegram에 테스트용 카드 수 건 도착
- 테스트가 건당 5~6초로 비정상 지연(네트워크 호출) — 이 지연이 발견 계기

## 근본원인 (확인됨)

테스트 fixture가 DB만 격리하고 **외부 서비스 자격증명은 격리하지 않았다**.
개발 편의로 실키를 로컬 .env에 두는 순간, "설정이 있으면 발송한다" 코드는 테스트에서도 발송한다.

## 수정

`tests/conftest.py`에 autouse 픽스처 `_isolate_external_credentials` 추가 — 외부
자격증명 환경변수 18종을 매 테스트 시작 전 빈 문자열로 강제 + settings 캐시 초기화.
특정 값이 필요한 테스트만 `settings_env(...)`로 그 테스트 안에서 재주입.

## 검증 결과

이후 전체 스위트(458개)에서 외부 호출 0건 확인. P4~P8의 신규 외부 연동(Resend·Meta·R2)
키도 같은 목록에 선제 등록돼 재발 차단.

## 재발방지 · 다른 프로젝트에도 적용할 규칙

[[test-isolation-of-external-credentials]] 패턴으로 일반화. embolos·aiwebbuilder처럼
로컬 .env에 실키(또는 prod DB)가 있는 저장소 전부가 대상.

## 관련 커밋과 문서

- 수정 커밋: fd1a21a (`backend/tests/conftest.py`의 `_EXTERNAL_CREDENTIAL_KEYS`)
