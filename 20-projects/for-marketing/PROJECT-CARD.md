---
type: project-card
schema_version: 2
project_id: for-marketing
status: active
repo_path: C:\Users\강지호\project\for-marketing
remote_url: https://github.com/StockHedge/for-marketing (비공개)
branch: main
head_commit: 8d61a1b
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

- 로컬 저장소: `C:\Users\강지호\project\for-marketing` (주: 조만간 다른 PC로 이관
  예정 — 기기 이관 시 이 필드와 `repo_path`를 함께 갱신할 것)
- 원격 저장소: `https://github.com/StockHedge/for-marketing` (비공개, gh CLI로
  2026-08-04 확인)
- 현재 상태: `docs/ai/NOW.md` (페이즈별 완료 기록·대기 항목)
- 접점 계약 정본: `docs/contracts/t9-integration.md`(플랫폼), `docs/contracts/dashboard-api.md`(대시보드 v7)
- 전체 설계: `~/.claude/plans/c-users-downloads-marketing-pipeline-ma-partitioned-kay.md` (승인본)

## 구조 지도

- backend/ FastAPI 단일 앱 — 6계층 디렉터리(collectors/intelligence/gates/executors/feedback/orchestrator)
  - 오케스트레이션: GitHub Actions cron → `POST /internal/cron/{job}` (X-Cron-Secret, advisory lock 멱등)
  - Meta 연동 4모드: disabled|stub|dryrun|live — 토큰 없이 전 구간 동작
    (2026-08-04부터 live, 페이지 토큰 보유 여부로 게이팅)
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

**P0~P9 전 페이즈 + 배포 + Meta live 전환 + P7 실 Graph 발행까지 완료(2026-08-04).**
이전 카드(커밋 9034e75, P0~P9 빌드 완료 시점) 이후 진전: 운영 배포(Fly+Neon+Upstash+
CF Pages, cron 전 경로 실증) 완주 / Meta 60일 사용자 토큰→무기한 페이지 토큰 교환 및
토큰 매니저(store/refresh/health) 신규 구현(P1 명세 누락분을 부트스트랩 시점에 발견) /
P7 실 Graph 발행·커뮤니티 클라이언트 전량 실구현 / R2 실연결 / Meta 웹훅 구독 active /
렌더링 에셋 저장소 편입 / GH Actions cron 100분 주기 통합. 상세는
[[2026-08-04-p7-live-publish-r2-webhook]]와 저장소 `docs/ai/NOW.md`.

남은 것: core 해시태그·경쟁 계정 목록, 발신 서브도메인 DNS(대량 발송 전), DoD ⑤
이메일 실도달(발송창 내 재실행), 광고 실집행 검증, `generate_enabled` 스위치 전환
(사용자 판단 대기), `ig_user_id` Meta 측 전파 대기(야간 잡 자동 재시도 중), embolos
트랙(2026-08-11 리마인드 예정).

## 중요 사건

- [[2026-08-03-tests-called-real-external-apis]] — 테스트가 실자격증명 상속으로 실제 외부 API 호출
- [[2026-08-04-secret-leaked-via-error-url]] — Meta 토큰 부트스트랩 오류 문자열의
  URL이 client_secret·토큰을 유출(운영 DB 1행 한정, resolved)

## 주요 마일스톤

- [[2026-08-04-hub-p0-p8-built]] — 허브 전 계층 구축 + 실연동 검증
- [[2026-08-04-p7-live-publish-r2-webhook]] — Meta live 전환 + P7 실 Graph 발행 +
  R2 실연결 + 웹훅 구독

## 관련 공통 패턴

- [[test-isolation-of-external-credentials]]
- [[s3-compatible-storage-silent-incompatibility]]
- [[gitignored-runtime-assets-break-deploy]]

## 마지막 검증

- 확인일: 2026-08-04 (P7·R2·웹훅 세션 — 이번 갱신은 이 세션 결과 반영)
- 확인한 근거: backend pytest 559 passed·ruff·mypy 클린 / frontend typecheck 통과 /
  셋업 체크리스트 실프로브 10개 중 9 ok(embolos만 미설정) / R2 실왕복(업로드→공개
  GET 200→회수→GET 404) / Meta 웹훅 구독 active 확인 / 운영 배포
  (Fly+Neon+Upstash+CF Pages) cron 전 경로 실증
- 확인하지 못한 항목: 광고 실집행(예산 소액이라도 live 집행 실증 없음), 이메일
  실도달(발송창 내 재실행), embolos 트랙, `ig_user_id` 바인딩(자동 재시도 진행 중)
