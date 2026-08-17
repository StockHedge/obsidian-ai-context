---
type: milestone
schema_version: 2
project: youth-investment
date: 2026-08-17
status: resolved
root_cause_status: confirmed
agents: [claude-code]
tags: [launch-gate, deploy, security]
---

# 2026-08-17 — 출시 게이트 완료 (단일 세션)

## 달성 내용

1. **로컬 동기화**: 3/24 정체 로컬을 origin/main 으로 정렬 (94커밋 격차 해소, venv 커밋 폐기).
2. **레거시 제거**: RN 0.73 구앱 2벌(`YouthInvestment/`, `mobile/`) + kiwoom 잔재 4종 + dead file.
3. **코드리뷰 High 14건 전수 처리**: 6건은 이전 PR 에서 기수정 확인, 나머지 실수정 —
   push batch(단일 쿼리+Expo 100 chunk), 랭킹 limit=None, 컬러 토큰 4종 신설, API timeout 15s.
4. **거래시간 가드 활성화**: `market_hours_enforced` 기본 true (fail-closed), 테스트 7건 추가.
5. **인증 스택 교체**: python-jose→PyJWT 2.10.1, passlib→bcrypt 4.2.1 직접, aioredis 제거.
   기존 $2b$ 해시 호환 유지.
6. **CI 자동 배포**: main push 트리거 활성 + 빌드 컨텍스트 버그 픽스 → 운영 배포 성공 검증.
7. **웹 프론트 운영 배포**: Expo web export → Cloudflare Pages `https://finple.pages.dev`.
   `ALLOWED_ORIGINS` 갱신(`fly secrets set` + `secrets deploy`) 후 CORS 왕복 실측.

## 검증 근거 (증거 우선순위 상위)

- pytest **135/135 PASS**, tsc **0 errors**
- GitHub Actions run 32031531952 (deploy) success
- 운영 실측: `/health` 200, preflight 200 + `access-control-allow-origin` 정확,
  비허용 origin 400 거부, 웹앱에서 `/api/auth/me` 401 정상 왕복

## 재사용 교훈 (30-patterns 승격 후보)

- `flyctl deploy --config <dir>/fly.toml` 을 리포 루트에서 실행하면 빌드 컨텍스트가
  루트가 되어 Dockerfile COPY 가 깨진다 — CI 에서는 `working-directory` 를 config 위치로.
- Fly staged secret 은 `fly apps restart` 로는 적용되지 않는다 — `fly secrets deploy` 필요.
- pydantic-settings 의 `List[str]` env 는 JSON 배열 형식이다 (쉼표 구분 아님).

## 추가 검증 (Android 에뮬레이터 실기, 같은 날)

- Expo Go(SDK 55)로 coop_td AVD 에서 앱 구동 — Splash·가입 3단계 렌더 정상.
- 가입 폼 → 운영 API 실연동 확인 (이메일·닉네임 중복확인 "사용가능").
- **결함 발견·수정**: 운영 phone/send 가 NCLOUD 미구성 가드 없이 SENS 호출 → 500.
  ServiceUnavailableError(503, SMS_NOT_CONFIGURED) 신설·배포 후 실기 재검증 —
  앱에 "휴대폰 인증(SMS)이 아직 준비되지 않았습니다" 정상 표시.
- 교훈: adb `input keyevent 111`(ESC)은 RN 화면 back 을 유발 — 키보드 닫기는 `4`(BACK).

## 남은 것

**회원가입이 SENS 키 등록 전까지 본인인증에서 차단** (열려면 SENS 등록 or 인증 생략
토글 — 정책 결정 필요). EAS 네이티브 빌드(계정은 stockhedges-team 로그인 확인됨),
웹 NetworkError 1건(/ws 추정), PR #69 rebase 결정, quiz v1/v2 통합 결정.
시크릿 회전은 사용자 결정으로 미진행.
