---
type: milestone
schema_version: 1
project: embolos
status: completed
created: 2026-08-04
work_date: 2026-08-04
agents: [claude-code]
verification: wiring-8 + ext-13(db-roundtrip) + isolation-anchors + catalog-21 + route-snapshot + ci-parity-green
source_repo: StockHedge/embolos + StockHedge/embolos-app
source_commit: b24d02c / 02495c3 / 5624d99 (backend) · acc5d4d (app)
tags: [analytics, ai-track, dual-write-parity, k-anonymity, llm-minimalism, external-ai-handoff]
---

# Embolos — AI 분석 트랙 P1~P3 완결 + 앱 트랙 마감

## 무엇이 끝났나

외부 설계 세션의 3-zip 계약(phase1/2/3)을 repo 세션이 검증·정정하며 전량 적용:

1. **P1 W2 개통**(`b24d02c`): 원시 이벤트 수집(event_tracking — _asid 쿠키·회전솔트
   visitor_hash·fail-open) + 핫패스 훅 8지점 + cron 4종(매시 재집계/일배치/파리티/90일 파기).
   구세대 카운터와 **이원 기록** — 파리티 2주 무위반 통과 전 구세대 은퇴 금지.
2. **P2 갭 스키마**(`02495c3`, 0077): 혜택·손님소리·메시지·예약 일별 + 액션 스냅샷 + 이상
   신호(순수 함수 판정·소표본 3중 가드) + 업종 벤치마크(쓰기 시점 k-익명 HAVING+스윕) +
   tenants.industry_code(온보딩 기록·백필 없음).
3. **P3 표면**(`5624d99`, 0078): 일일 다이제스트(LLM 0원·무소식 스킵·email opt-in/push 기본
   on), `/seller/ai` AI 분석 센터, 주간 AI 리포트(haiku 단발 JSON·컨텍스트 원문 영속=재현성
   계약·유료 게이트). 트리거 카탈로그 57종.
4. **앱 트랙 마감**(`acc5d4d`): PDP 재입고·문의/FAQ·리뷰 투표, 스토어 네비, 셀러 예약 2화면.

## 재사용 가능한 교훈

- **외부 AI 산출물: diff 클린 적용 ≠ 동작.** asyncpg 타입 추론 결함 2건(같은 파라미터의
  이중 문맥 → AmbiguousParameterError / date-int 추론 실패)이 fail-open 경로에 숨어 있어
  미봉합 시 **전 이벤트 무손 유실**이었다. DB 왕복 테스트(같은 키 2회 기록 → pageviews·
  max_stage 검증)가 잡았다. 원격 계약 zip에는 반드시 왕복 테스트를 붙여 받을 것.
- **A-가정(추론 식별자) 검증표 방식이 유효**: 설계 세션이 가정을 A1~A9로 명시·국소화해 준
  덕에 repo 세션의 실코드 대조가 시간당 수 건 속도로 끝났다. 6/9가 부분·전면 오류(테이블명·
  컬럼명·상태 어휘·정책식)였다 — 가정 명시가 없었으면 조용한 프로덕션 결함이 됐을 것.
- **파라미터 캐스트 관례**: raw SQL에서 같은 named param을 두 문맥(INSERT 대상 + WHERE 비교,
  date 연산)에 쓰면 asyncpg가 prepared statement에서 거부한다 — `(:p)::text`/`(:p)::int`
  명시 캐스트를 관례화.

## 남은 것

~~prod 재배포(0075~0078)~~ → **완료(같은 날 v37 — 후속 노트
[[2026-08-04-push-custom-fields-benefit-backlog]] 참조)**. 잔여: 실기기 푸시 종단 검증(사용자
휴대폰) → main 병합(cron 발화 조건) → 파리티 2주 관찰 → refresh_legacy 전환. 정본은 repo
`docs/ai/NOW.md`.
