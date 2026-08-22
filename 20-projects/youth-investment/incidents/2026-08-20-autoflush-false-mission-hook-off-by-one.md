---
type: incident
schema_version: 1
project: youth-investment
component: backend / mission-engine
category: correctness
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-20
agents: [claude-code]
source_repo: StockHedge/YouthInvestment
tags: [sqlalchemy, autoflush, read-your-own-writes, off-by-one, test-prod-parity]
---

# autoflush=False 세션에서 이벤트 훅이 항상 한 이벤트 늦게 평가됐다

## 요약

주문 체결 직후 호출되는 미션 평가 훅(`on_order_filled`)이 **자기 자신의 주문을
보지 못했다**. LIMIT 첫 체결이 ORDER_LIMIT_USE 로 카운트되지 않고, DIVERSIFY_5
진행도가 직전 주문 시점 값(4/5)으로 저장되고, FIRST_BUY 토스트가 첫 매수가 아닌
**두 번째 매수에서** 떴다. 다음 이벤트에서 소급 완료되기 때문에 "가끔 안 됨"
류의 은폐형 결함이었다.

## 근본 원인

운영 세션이 `async_sessionmaker(autoflush=False)` — 훅의 평가 SELECT 가 같은
트랜잭션의 **pending 변경(방금 FILLED 된 주문·포지션)을 flush 없이는 못 본다**.
미션 엔진은 "훅 시점 DB = 최신 상태"를 암묵 전제해 계약이 어긋났다.

이 결함이 테스트를 통과해 온 이유: **conftest 의 세션은 autoflush 기본값(True)**
이라 운영과 설정이 달랐고, 기존 테스트들은 셋업마다 명시 flush 를 호출해
우연히도 결함 조건을 만들지 않았다.

## 수정

훅 호출부마다가 아니라 **미션 엔진 진입점(`_evaluate_codes`) 에서 flush 1회** —
"평가는 최신 상태 전제"라는 계약을 엔진이 스스로 보장. 회귀 테스트는 운영과
동일한 autoflush=False 세션으로 pending 주문을 재현, 픽스 제거 시 FAIL / 적용 시
PASS 를 확인 (`89f9151b`).

## 재사용 교훈

- **테스트 세션 설정은 운영 설정과 동일해야 한다** (autoflush, expire_on_commit,
  isolation). 다르면 read-your-own-writes 류 결함이 테스트를 영원히 통과한다.
- 같은 트랜잭션의 상태를 읽는 이벤트 훅/사이드이펙트 계층은 진입점에서 flush 로
  전제를 보장하라. 호출부마다 flush 를 요구하는 설계는 반드시 누락된다.
- 증상이 "항상 한 이벤트 지연"이면 autoflush/flush 시점부터 의심하라.
- 관련: [[2026-08-20-alembic-stamp-seed-loss]]
