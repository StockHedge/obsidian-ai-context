---
type: incident
schema_version: 1
project: for-marketing
component: ci / orchestrator-cron
category: cost-and-infrastructure
severity: low
status: resolved
root_cause_status: confirmed
discovered: 2026-08-11
resolved: 2026-08-11
verified: 2026-08-11
agents: [claude-code]
source_repo: StockHedge/for-marketing
tags: [github-actions, cron, cloudflare-workers, cost, orchestrator]
---

# 크론 트리거를 Cloudflare Worker로 이관 — 이 저장소는 가해자가 아니라 연루자였다

## 요약

계정 전체 GitHub Actions 무료 한도가 2026-08-11에 소진됐고, 이 저장소는 **313분(15.9%)**을
썼다. `scheduled-jobs.yml`의 크론 7종을 Cloudflare Worker(`StockHedge/ops-cron`)로 옮겼다.

## 이 저장소는 자기 예산을 지켰다

`scheduled-jobs.yml` 주석에 *"비용: 무료 2,000분/월 內 운영이 목표(월 ~700분 추정)"* 라고
적혀 있고, 실측은 **월 환산 476분**으로 목표 안이었다. 2026-08-04에 15분 레인과 매시 레인을
100분 레인으로 통합한 비용 결정도 그대로 유효했다(실측 발화율 73%로 설정에 근접).

**문제는 이 저장소가 아니라 구조였다.** 한도는 계정 단위인데 예산은 저장소 단위로 세웠고,
embolos(56%)·aiwebbuilder(11%)·contest(11%)와 합산되자 한도를 넘었다.
이 교훈은 [[ci-cron-trigger-billing-trap]] 에 승격했다.

## 이관 내용

`scheduled-jobs.yml`의 크론 7종 전부. 러너에서 실연산 없이 `curl -X POST` 만 하던
구조라 이관에 제약이 없었다.

- **100분 레인 3개 엔트리는 하나의 레인**이다. cron이 "매 100분"을 단일식으로 표현하지
  못해 5시간 주기 3개로 구현한 것 — Worker 쪽 `src/schedule.js` 에도 이 사실을 주석으로
  명시했다. 쪼개면 안 된다.
- `manual` job(`workflow_dispatch`로 임의 잡 실행)은 **이관하지 않았다.**
  공개 URL로 임의 엔드포인트를 때릴 수 있으면 안 되기 때문이다.
- `ci`(빌드·테스트, 월 319분 추정)는 실제 러너 연산이라 Actions에 남는다.

## 주기는 유지하기로 했다

전체 크론 주기 완화를 검토하면서 이 저장소의 100분 레인을 200분으로 늦추는 안이 있었으나
**기각**했다. 레인 안에 `telegram_poll`이 있다 — 사람이 텔레그램 버튼을 누른 것에 반응하는
경로다. 200분이면 승인 후 최대 3.3시간 뒤에 반영되어 사람이 쓰는 경로가 고장 난 것처럼
느껴진다. 주석이 이미 "승인 버튼 반영 최대 100분"을 감수 비용으로 명시했고, 그것이 상한이다.

`registry.py`의 `expect_max_s`는 **실행 시간** 기대치이지 주기가 아니고, dead-man 워처는
"향후(P3+)"로 미구현이라 주기 변경에 코드 결합은 없다. (embolos는 결합이 있어서 다르다 —
[[2026-08-11-actions-quota-and-cron-migration]] 참조)

## 남은 문제 — 실패율 50%

이번 청구 주기에 `scheduled-jobs` 실행 **161회 중 80회가 실패**했다. 실패해도 과금은 된다.
Worker로 옮긴 뒤에는 텔레그램으로 실패 알림이 가지만, **실패율 자체가 비정상**이다.
어느 step에서 깨지는지, 백엔드 문제인지 시크릿·타임아웃 문제인지 확인이 필요하다. **미해결**.

## 문서 갱신 필요

`backend/app/orchestrator/registry.py` 상단 docstring에
*"실제 실행 스케줄은 `.github/workflows/scheduled-jobs.yml`이 정본이다"* 라고 적혀 있는데,
이관 후 **거짓**이 됐다. 정본은 `StockHedge/ops-cron` 의 `src/schedule.js` 다.

## 롤백

원본 워크플로는 삭제하지 않고 비활성화만 했다. 재활성화 한 번으로 복구되며, 되돌릴 때는
Worker 쪽 크론을 먼저 멈춰야 이중 실행이 없다. 절차는 `ops-cron/docs/ROLLBACK.md`.
