---
type: project-card
schema_version: 2
project_id: stockhedge-landing
status: active
repo_path: (사용자 홈)\TheNewProject\StockHedgeLanding
remote_url:
branch: main
head_commit: 4065409
updated: 2026-09-01
last_verified: 2026-09-01
agents: [codex]
tags: [company-site, astro, cloudflare-pages, dns, email-routing]
---

# StockHedge 회사 홈페이지

## 한 줄 목적

StockHedge를 디지털 제품 스튜디오로 소개하고 AI웹빌더·FinPle 및 향후 사업을 연결하는
한국어 중심의 공식 회사 홈페이지를 운영한다.

## 기준 위치

- 로컬 저장소: `(사용자 홈)\TheNewProject\StockHedgeLanding` (`main`)
- 원격 저장소: 없음 — 사용자의 결정에 따라 로컬 Git만 사용한다.
- 현재 상태: 저장소 `docs/ai/NOW.md`
- 운영 사이트: https://stockhedge.kr
- Cloudflare Pages 프로젝트: `stockhedge-landing`

## 구조 지도

- `src/` — Astro 페이지·레이아웃·스타일과 정적 Venture 데이터
- `public/` — 브랜드 자산, favicon, OG 이미지, robots 및 보안 헤더 설정
- `tests/` — Playwright 운영 스모크 테스트
- `DESIGN.md` — Industrial Product Registry 비주얼 방향과 디자인 토큰
- `docs/ai/NOW.md` — 현재 운영 상태와 남은 검증

## 핵심 제약

- canonical 도메인은 `https://stockhedge.kr` 하나로 유지한다.
- `www`와 `stockhedge-landing.pages.dev`는 경로·쿼리를 보존해 루트 도메인으로 301 전환한다.
- 세 번째 사업 슬롯은 이름·설명·링크 없이 `03`과 `IN DEVELOPMENT` 상태만 노출한다.
- `support@stockhedge.kr`, `official@stockhedge.kr`은 수신 전달 주소이며,
  `noreply@stockhedge.kr`과 catch-all은 Drop한다.
- Workers Free에서는 Cloudflare Email Sending을 사용할 수 없다. 유료 전환 또는 별도
  발신 사업자 선택 전에는 SMTP/API 발신을 구성하지 않는다.
- 목적지 메일 주소, API 토큰, SMTP 자격증명 등 비밀·개인정보를 저장소와 Vault에 기록하지 않는다.

## 현재 단계

**운영 공개 완료, 메일 실수신 검증 대기 (2026-09-01)**

- Astro 정적 사이트를 Cloudflare Pages에 배포하고 가비아 네임서버를 Cloudflare로 전환했다.
- 루트·`www`·Pages 도메인, HTTPS, SEO 메타데이터, 보안 헤더, 404를 운영 환경에서 검증했다.
- Email Routing의 support/official 전달, noreply/catch-all Drop과 MX·SPF·DKIM을 구성했다.
- 가비아에 DNSSEC DS/KEYDATA를 등록했고 공용 DNS에서 DS·DNSKEY·RRSIG를 확인했다.
- 남은 작업은 외부 메일함을 이용한 실제 수신/Drop 테스트와 Cloudflare DNSSEC 화면의
  상태 표시 갱신 확인이다.

## 중요 사건

- 2026-09-01: `stockhedge.kr` 공식 홈페이지와 도메인·메일 수신 기반을 한 번에 개통했다.
- 2026-09-01: Email Sending은 Workers Paid 전용임을 대시보드와 CLI에서 확인했다.
  계획의 확정 가정대로 임의 유료 전환이나 다른 사업자 도입 없이 차단 사항으로 남겼다.

## 주요 마일스톤

- **2026-09-01: 공식 사이트·도메인·Email Routing 개통** —
  [[milestones/2026-09-01-company-site-domain-email-routing-launch]] 참조.

## 관련 공통 패턴

- (미기록) 등록기관에서 DNSSEC KEYDATA까지 요구하는 경우 Cloudflare DS 값과 공개키를
  함께 등록하고, 대시보드 표시보다 공용 DNS의 DS/DNSKEY/RRSIG 체인을 우선 검증한다.

## 마지막 검증

- 확인일: 2026-09-01
- 확인한 근거: `astro check`, 프로덕션 빌드, Playwright 9개 테스트 통과 기록;
  운영 루트 200 및 보안 헤더; `www`·Pages 경로/쿼리 보존 301; 공용 DNS DS
  `2371 / algorithm 13 / digest type 2`; canonical·OG·구조화 데이터·robots·sitemap·404.
- 확인하지 못한 항목: support/official의 실제 Naver 받은편지함 도착, noreply/catch-all의
  외부 발송 Drop, Cloudflare 대시보드 DNSSEC 상태 표시 갱신, 유료 Email Sending 발신.

