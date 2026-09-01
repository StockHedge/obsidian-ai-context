---
type: milestone
schema_version: 2
project: stockhedge-landing
date: 2026-09-01
status: monitoring
root_cause_status: confirmed
agents: [codex]
tags: [launch, astro, cloudflare-pages, dns, dnssec, email-routing]
---

# 2026-09-01 — 공식 사이트·도메인·Email Routing 개통

## 달성 내용

1. **회사 홈페이지 구현**: Astro·TypeScript 정적 원페이지로 StockHedge를 디지털 제품
   스튜디오로 소개했다. AI웹빌더와 FinPle을 연결하고 세 번째 사업은 비어 있는
   `IN DEVELOPMENT` 슬롯으로 모델링했다.
2. **브랜드·접근성·SEO**: Industrial Product Registry 방향, Pretendard Variable과
   IBM Plex Mono, 회색 그리드와 산성 라임 액센트를 적용했다. canonical, Organization
   구조화 데이터, sitemap, robots, favicon, 1200×630 OG 이미지와 보안 헤더를 추가했다.
3. **운영 배포**: Cloudflare Pages `stockhedge-landing`에 배포하고 가비아의
   `stockhedge.kr` 네임서버를 Cloudflare로 전환했다.
4. **도메인 정규화**: `stockhedge.kr`을 canonical로 두고 `www.stockhedge.kr`과
   `stockhedge-landing.pages.dev`를 경로·쿼리 보존 301 리디렉션으로 통합했다.
5. **수신 이메일 구성**: `support@`·`official@`은 검증된 목적지 받은편지함으로 전달하고,
   `noreply@`와 catch-all은 Drop하도록 Cloudflare Email Routing을 설정했다.
6. **DNS·DNSSEC**: Email Routing MX/SPF/DKIM을 확인하고, 가비아에 Cloudflare DNSSEC의
   DS와 KEYDATA를 등록했다. 공개 DNS의 DS·DNSKEY·RRSIG로 체인을 확인했다.
7. **발신 범위 확정**: Workers Free 요금제에서는 Email Sending이 제공되지 않음을 확인했다.
   사용자 승인 없는 유료 업그레이드나 대체 메일 사업자 전환은 수행하지 않았다.

## 주요 결정

- 공개 주소는 `stockhedge.kr`을 그대로 사용하며 별도 사이트 도메인을 만들지 않는다.
- 워드마크는 타이포그래피로만 구성하고 별도 심볼 로고는 만들지 않는다.
- 문의 CTA는 `official@stockhedge.kr`, 고객지원은 `support@stockhedge.kr`, 대표 전화는
  `010-7666-4510`을 사용한다.
- 문의 폼·CMS·분석 스크립트는 이번 범위에서 제외한다.
- 로컬 Git 저장소만 유지하고 별도 GitHub 원격 저장소는 만들지 않는다.

## 검증 근거

- 로컬: `astro check`, 프로덕션 빌드, Playwright **9개 테스트 통과**.
- 브라우저: 375px·768px·1280px에서 수평 오버플로, 내비게이션, 한글 줄바꿈,
  카드 배열과 푸터를 확인했다. 빈 세 번째 카드는 포커스·클릭되지 않는다.
- 운영 루트: `https://stockhedge.kr/` **200**, HSTS·CSP·Permissions-Policy·
  Referrer-Policy·X-Content-Type-Options·X-Frame-Options 확인.
- 리디렉션: `www`와 Pages 도메인의 `/ventures?src=...` 요청이 동일 경로·쿼리를 보존해
  `https://stockhedge.kr/ventures?src=...`로 **301**.
- 오류·검색 노출 기반: 존재하지 않는 경로 **404/no-store**, `robots.txt`와
  `sitemap-index.xml` **200**, canonical·OG·Organization JSON-LD 확인.
- DNSSEC: 공개 DS의 key tag **2371**, algorithm **13**, digest type **2**와 DNSKEY·RRSIG 확인.
- Git 기준점: 로컬 `main`의 `4065409` (`docs: 공개 도메인과 이메일 설정 기록`), 원격 없음.

## 남은 검증

- 외부 메일함에서 `support@stockhedge.kr`, `official@stockhedge.kr`로 보내 실제 목적지
  받은편지함 도착을 확인한다.
- `noreply@stockhedge.kr`과 임의 주소를 대상으로 Drop을 확인한다.
- Cloudflare DNSSEC 화면의 보류 표시가 공개 DNS 상태에 맞게 갱신되는지 확인한다.
- 사람 답장·시스템 발신은 Workers Paid Email Sending을 명시적으로 활성화하거나 별도
  발신 사업자를 선택한 뒤 SPF·DKIM·DMARC와 실제 발송을 검증한다.

## 보안 메모

목적지 받은편지함 주소, 인증번호, API 토큰, 쿠키, SMTP 자격증명은 이 기록에 남기지 않았다.

