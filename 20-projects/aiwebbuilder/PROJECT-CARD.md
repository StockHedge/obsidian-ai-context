---
type: project-card
schema_version: 2
project_id: aiwebbuilder
project: AI Webbuilder
status: active
repo_path: (사용자 홈)\projectaiwebbuilder
updated: 2026-08-09
last_verified: 2026-07-31
agents: [claude-code]
tags: [aiwebbuilder, marketing, saas]
---

# AI Webbuilder — 프로젝트 카드

## 한 줄 목적

한국 시장 특화 AI 웹사이트 빌더(소상공인·프리랜서 대상). 유형을 고르고 폼을 채우면 AI가
5~10분 안에 사이트를 만들어 배포한다. 미리보기 무료, 결제는 마음에 들 때만.

## 기준 위치

- 제품 구현 상세: 프로젝트 저장소의 `CLAUDE.md`·`PROGRESS.md`가 기준이다.
  이 Vault의 기록과 충돌하면 **코드가 이긴다.**
- 마케팅 실행 기준: [[AIWebbuilder]] (MOC)
- 새 세션이 가장 먼저 읽어야 할 문서: [[확정-결정]]

## 구조

이 프로젝트는 마케팅 지식 베이스라서 `incidents/ · milestones/ · retrospectives/` 대신
주제별 MOC 구조를 쓴다. [[20-projects/README|20-projects 구조 규약]]에 대한 의도된 예외다.

- `00-핵심/` — 확정 결정과 반복해서 데인 함정
- `10-캠페인/` — 캠페인 1건 = 노트 1개
- `20-메시지/` — 카피·톤과 표기 주의사항 (허위광고 리스크)
- `30-자산/` — 포스터·목업·Figma 위치와 최신본
- `40-가격·프로모션/` — 가격표와 플래그 지도
- `50-실행기록/` — 날짜별 실행·변경 로그

## 알려진 리스크

- 광고 문구와 실제 코드가 갈라진 전례가 반복됐다(티어 표기, 만료 개념, 기한 하드코딩).
  금전 혜택을 다루므로 분쟁 소지가 실질적이다. 광고 집행 전 [[표기-주의사항]]을 먼저 본다.

## 마지막 검증

- 확인일: 2026-07-31 (MOC 기준)
- 확인한 근거: 백엔드 설정·가격·프로모션 API·i18n 로케일 직접 대조 + 백엔드 테스트 통과
- 확인하지 못한 항목: 운영 DB `app_settings` 실값, 포스터 QR 실제 스캔,
  START2026의 DB `expires_at` 반영 여부
- 2026-08-09: 노트북 Vault에서 이 Vault로 통합. 파일·폴더명은 내부 위키링크 보존을 위해
  원본 그대로 유지했다 ([[VAULT-STATUS]] 「알려진 제약」 참조).
