---
type: project-card
schema_version: 2
project_id: for-marketing
status: active
repo_path: C:\Users\강지호\project\for-marketing
remote_url: (없음 — 로컬 전용, 원격 생성은 사용자 결정 대기)
branch: main
head_commit: 9034e75
updated: 2026-08-04
last_verified: 2026-08-04
agents: [claude-code]
tags: [marketing, automation, dashboard, fastapi, react]
---

# for-marketing — 마케팅 자동화 허브

## 한 줄 목적

aiwebbuilder·embolos 두 플랫폼의 마케팅(수집→생성→게이트→발행→광고→피드백)을
단일 대시보드에서 자동화·관제한다. 북극성 지표는 좋아요가 아니라 가입·결제 전환.

## 기준 위치

- 로컬 저장소: `C:\Users\강지호\project\for-marketing`
- 현재 상태: `docs/ai/NOW.md` (페이즈별 완료 기록·대기 항목)
- 접점 계약 정본: `docs/contracts/t9-integration.md`(플랫폼), `docs/contracts/dashboard-api.md`(대시보드 v7)
- 전체 설계: `~/.claude/plans/c-users-downloads-marketing-pipeline-ma-partitioned-kay.md` (승인본)

## 구조 지도

- backend/ FastAPI 단일 앱 — 6계층 디렉터리(collectors/intelligence/gates/executors/feedback/orchestrator)
  - 오케스트레이션: GitHub Actions cron → `POST /internal/cron/{job}` (X-Cron-Secret, advisory lock 멱등)
  - Meta 연동 4모드: disabled|stub|dryrun|live — 토큰 없이 전 구간 동작(현재 stub)
- frontend/ React SPA — 홈/파이프라인(칸반)/승인/소재실/광고/이메일/커뮤니티/어트리뷰션/설정
- 로컬 기동: `docker compose up -d`(pg:5434, redis:6381) + backend uvicorn:8000 + `npm run dev`:5173

## 핵심 제약 (운영 수칙)

1. **가드레일은 코드 상수, 설정으로 완화 불가**: 광고 일 30,000/캠페인 15,000원, 게시 일 10건,
   이메일 발송창 09~20 KST·신고율 0.3% 전면 차단. `effective()=min()` — 엄격한 방향으로만 조정 가능.
2. **예산 증액·캠페인 생성은 API/UI 자체가 없음** — Telegram/콘솔 승인(T2) 경유가 유일 경로.
3. 외부 부작용 choke point 3개(send_email/apply_ad_action/publish)만 존재. 우회 경로 금지.
4. auto_pass(T0) 기본 OFF — 가동 첫 2주 전량 사람 승인(마스터 플랜 §10).
5. 시크릿은 backend/.env(로컬)·fly secrets(운영)만. Vault·git·문서 기록 금지.
6. 두 플랫폼 저장소 수정은 최소 침습 — 규약은 저장소 `CLAUDE.md` "타 저장소 수정 시 규약" 절.

## 현재 단계

**P0~P9 전 페이즈 완료(2026-08-04)** — 멀티에이전트 전체 리뷰(확정 20건 전량 수정) 포함.
pytest 510/프론트 58 그린. 남은 것은 전부 외부 의존(배포·Meta 토큰·실데이터 시드) —
상세는 저장소 `docs/ai/NOW.md`.

대기 중 사용자 액션: Meta 페이지 토큰(P7~P8 live 전환), YOUTUBE_API_KEY·META_APP_ID/SECRET(실수집),
core 해시태그·경쟁 계정 목록, 발신 서브도메인 DNS(대량 발송 전), DoD ⑤ 실도달(발송창 내 재실행).

## 중요 사건

- [[2026-08-03-tests-called-real-external-apis]] — 테스트가 실자격증명 상속으로 실제 외부 API 호출

## 주요 마일스톤

- [[2026-08-04-hub-p0-p8-built]] — 허브 전 계층 구축 + 실연동 검증

## 관련 공통 패턴

- [[test-isolation-of-external-credentials]]

## 마지막 검증

- 확인일: 2026-08-04
- 확인한 근거: backend pytest 458 passed·ruff·mypy 클린 / frontend 테스트 54·typecheck·build 클린 /
  브라우저 실연동 E2E(획득·어트리뷰션·승인·발행 stub·알림 Telegram 실도착)
- 확인하지 못한 항목: Meta live 발행·광고 실집행(토큰 대기), 이메일 실도달(발송창), 배포 환경
