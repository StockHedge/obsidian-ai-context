---
type: incident
schema_version: 1
project: embolos
component: ci / cron / ops-watchers / billing-secret
category: cost-and-infrastructure
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-11
resolved: 2026-08-11
verified: 2026-08-11
agents: [claude-code]
source_repo: StockHedge/embolos
tags: [github-actions, cron, cloudflare-workers, secrets-rotation, dead-man, cost]
---

# GitHub Actions 한도 소진 → 크론 Cloudflare Worker 이관 + 크론 시크릿 2종 강제 로테이션

## 요약

계정 전체 GitHub Actions 무료 한도(2,000분/월)를 **10.5일 만에 100% 소진**했고,
**embolos가 그중 56%(1,110분)**를 차지했다. 원인은 연산량이 아니라 과금 단위였다.
러너에서 실연산 없이 `curl` 만 하던 크론을 Cloudflare Worker(`StockHedge/ops-cron`)로 옮겼다.

이관 과정에서 `CRON_SECRET`·`BILLING_CRON_SECRET` 값을 **어디서도 복구할 수 없어
강제 로테이션**했고, dead-man 워처가 크론 주기를 하드코딩하고 있다는 사실을 발견했다.

## 무엇이 비쌌나

`scheduled-jobs.yml` 단독으로 562분(전체 한도의 28%).

```
job 11개 중 9개 skipped, 실제 실행은 2개
실 연산: ops-watchers 13초 + frequent 8초 = 21초
과금:   2 job × 1분 올림       = 2분
```

Actions는 run이 아니라 **job 단위로 분 단위 올림** 과금이다. 375회 반복해 562분이 됐고
실 연산 대비 효율은 17%였다. `tests`(360분)는 실제 pytest 실행이라 정당한 비용이고
Actions에 남겼다.

## 근본원인 2 — 시크릿이 .env에 없었다

`CRON_SECRET`·`BILLING_CRON_SECRET` 값을 복구하려 했으나 실패했다.

- Fly secrets: **write-only** (다이제스트만 조회 가능)
- GitHub Actions Secrets: **write-only**
- `backend/.env`: **이 두 키만 누락** (다른 26개 키는 전부 있었음)
- PowerShell 히스토리: 없음

결과적으로 **머니패스 자격증명(`BILLING_CRON_SECRET`)을 강제 로테이션**해야 했다.
`Fly → GitHub Actions → Cloudflare` 순서로 심어 403 창을 수 분으로 최소화했다
(Fly를 먼저 바꿔야 창이 짧다 — 역순이면 Fly 재배포 30~60초 내내 403).

**조치**: 두 키를 `backend/.env`에 기록했다(gitignore 적용 확인). 다음엔 로테이션 불필요.

## 발견 — dead-man이 크론 주기를 하드코딩하고 있다

`backend/app/ops/watchers.py`의 `JOB_EXPECTATIONS`가 **"외부 GH Actions 주기와 맞춤"**
주석과 함께 잡별 기대 주기를 갖고, `마지막 성공 < 기대주기 × 1.5`면 P2 경보를 낸다.

| 잡 | 기대 | 경보 임계 |
|---|---|---|
| `reconcile-pending` / `reconcile-closed` | 20분 | 30분 |
| `flush-deferred` / `push-receipts` / `analytics-rollup-recent` | 70분 | 105분 |
| 일배치 계열 | 26시간 | 39시간 |

**따라서 크론 주기를 바꾸려면 이 딕셔너리도 함께 고쳐 배포해야 한다.**
주기 완화 검토 중 `frequent`를 `*/30`으로 늦추려던 안이 있었으나, 임계가 정확히 30분이라
지터만으로 오탐이 났을 것이다. 최종적으로 **embolos 주기는 전부 현행 유지**로 결정했다.

### 미해결 — `run-charges` 기대치 불일치 (기존 문제)

`run-charges`의 기대 주기가 `26 * 3600`(26시간)인데 **실제 크론은 월 1회**(`17 3 1 * *`)다.
임계 39시간을 넘으므로 매월 2일부터 다음 달 1일까지 dead-man이 상시 stale 판정을 한다.
`emit_alert`의 fingerprint 중복 억제로 알림이 쏟아지진 않으나 경보가 상시 켜진 상태다.
이번 이관과 무관한 선행 문제이며 **미해결**이다.

## 최종 상태

- 크론 정본: `StockHedge/ops-cron` 의 `src/schedule.js`
- Cron Trigger는 `* * * * *` 1개만 사용(무료 한도가 계정당 5개). 매분 깨어나 내부 분기하므로
  원본 cron 식이 그대로 보존된다 — `17 3 1 * *` 같은 5분 비정렬 스케줄도 시각 불변
- **embolos 주기는 전부 원본 유지** (dead-man 결합 때문)
- `health-ping.yml`은 **이관하지 않고 폐기**. scale-to-zero 앱을 5분마다 깨워 살아있는지
  확인하는 것은 자기모순이고, 앱이 죽으면 다른 크론들이 실패해 텔레그램으로 이미 알린다.
  `JOB_EXPECTATIONS`에 없어 dead-man 오탐과도 무관하다
- 원본 워크플로는 삭제하지 않고 비활성화 — 롤백은 재활성화 한 번

## 검증

- cron 매처 테스트 56건 통과 (원본 cron 식 전수 + 머니패스 월 1회 보장 포함)
- 실제 발화 관찰로 `BILLING`은 성공, `CRON`은 403을 잡아냈다 → 로테이션으로 해소
- **단계적 컷오버가 값을 했다.** 원본을 먼저 껐다면 `X-Cron-Secret` 계열 7종이
  조용히 죽은 채 운영됐을 것이다

## 교훈

[[ci-cron-trigger-billing-trap]] 로 승격했다. 핵심은 두 가지다.

1. CI 러너는 컴퓨트 실행기이지 크론 스케줄러가 아니다. `checkout` 없이 `curl` 만 하는
   워크플로는 트리거 전용 인프라로 보낸다.
2. 외부에서 주입하는 시크릿도 로컬 `.env`(gitignore)에 사본을 둔다.
   "어차피 Fly에 있으니까"는 조회 불가라 사본이 아니다.

관련: [[secrets-oauth-ops]]
