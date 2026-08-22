---
type: incident
schema_version: 1
project: youth-investment
component: backend / db-init
category: data-loss
severity: high
status: resolved
root_cause_status: confirmed
discovered: 2026-08-20
agents: [claude-code]
source_repo: StockHedge/YouthInvestment
tags: [alembic, stamp, seed, migration, silent-failure, graceful-degradation]
---

# alembic stamp head 가 마이그레이션 안의 데이터 시드를 통째로 유실시켰다

## 요약

운영 미션 시스템이 **조용히 전부 무동작**이었다. 미션 화면은 "오늘의 미션 준비 중"
빈 상태, `GET /api/missions` 는 빈 배열, FIRST_BUY 등 어떤 미션도 적립되지 않았다.
에러 로그는 한 줄도 없었다.

## 근본 원인

운영 빈 DB 초기화 경로(`scripts/init_db.py`)가 `Base.metadata.create_all` +
`alembic stamp head` 방식이었다. **`stamp` 는 버전 표시만 하고 마이그레이션을
실행하지 않으므로**, 마이그레이션 007/020 안에 들어 있던 미션 18종 `bulk_insert`
가 한 번도 실행되지 않았다. `_seed_essentials` 는 savings·instruments·education·
profile 만 보완하고 missions 를 빠뜨렸다.

침묵의 두 번째 층: 미션 엔진이 graceful 설계(훅 실패 시 로그만)라서, Mission row
부재가 "평가 대상 없음 → 조용히 skip"으로 처리돼 **결함이 기능 침묵으로 은폐**됐다.

## 수정

`seed_data_sync.py` 가 마이그레이션 모듈을 importlib 로 로드해 **시드 상수를 단일
원천으로 재사용**(복제 금지)하며 idempotent 보완. release_command 경유 자동 반영.
회귀 테스트: 시드 2회 실행 후 18종 존재·중복 없음 (`83d469c2`).

## 재사용 교훈

- **시드를 마이그레이션에 넣는 설계 + stamp 초기화 경로 = 시드 유실.** 빈 DB 를
  create_all+stamp 로 초기화하는 프로젝트는 마이그레이션 내 데이터 INSERT 를
  전수 조사해 별도 시드 경로에 복제 없이(상수 로드) 편입해야 한다.
- graceful degradation 은 결함을 숨긴다. "조용히 skip" 경로에는 최소 1회
  warning 로그(예: "활성 미션 0건")를 남겨야 운영에서 발견 가능하다.
- 관련: [[2026-08-20-autoflush-false-mission-hook-off-by-one]]
