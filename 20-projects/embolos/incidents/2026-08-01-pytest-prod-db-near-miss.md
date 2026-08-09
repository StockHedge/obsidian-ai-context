---
type: incident
schema_version: 1
project: embolos
component: test-infra / local-env
category: data-loss-near-miss
severity: high
status: resolved
root_cause_status: confirmed
discovered: 2026-08-01
resolved: 2026-08-01
verified: 2026-08-01
agents: [claude-code]
source_repo: StockHedge/embolos
source_commit: 88c9d4f
tags: [pytest, prod-db, env-sourcing, shell-chain, structural-guard]
---

# pytest가 prod DB로 향한 니어미스 — 체인 단락으로 env 소싱 유실

## 요약

`ruff check … && SP=…; . "$SP/test-branch.env"` 형태의 셸 체인에서 ruff가 실패하자
`&&` 단락으로 `SP` 할당이 건너뛰어졌고, `. "/test-branch.env"`가 조용히 실패한 채
pytest가 로컬 `.env`(= **prod DB**)로 실행됐다. 시드 테넌트가 prod에 INSERT됐다.

## 피해

**오염 0건 (실측).** teardown(FK CASCADE DELETE)이 같은 세션에서 시드를 삭제했고,
prod READ ONLY 점검으로 잔존 0건·alembic `0060` 불변을 확인했다. prod 리비전이
뒤처져 있어(`products.notice_category` 부재) 시드 단계에서 즉시 에러로 드러났다 —
**스키마가 같았다면 침묵 진행했을 사고**라는 점이 핵심 교훈.

부수 노출: 마스킹 sed가 `URL=`만 커버해 `DATABASE_URL_DIRECT=`(테스트 브랜치
자격증명)가 채팅 로그에 1회 노출됐다(prod 아님, git·문서 기록 없음).

## 근본 원인

환경 규율("test-branch.env를 소싱하라")이 **사람/모델의 주의**에 의존했고, 명령
체인 한 조각의 실패가 그 규율을 조용히 무너뜨렸다. `.`(source)의 실패가 후속
명령을 막지 않는 것도 공범.

## 수정 (구조적 차단)

- `backend/tests/conftest.py` 최상단: `PYTEST_DB_OK` env(scratchpad
  `test-branch.env`에만 실리는 opt-in 마커)가 없으면 `SystemExit` — 커밋 `88c9d4f`.
- 실행 관례: env 소싱은 체인 **맨 앞** + `|| exit`, 린트 등과 `&&`로 묶지 않는다.
- 마스킹 sed는 `(PASSWORD|SECRET|KEY|URL[A-Z_]*)=`로 접미 변형까지.

## 일반화

[[env-sourcing-optin-guard]] (30-patterns/ai-reliability) 참조 — "위험한 기본값 +
opt-in 마커" 패턴으로 승격.
