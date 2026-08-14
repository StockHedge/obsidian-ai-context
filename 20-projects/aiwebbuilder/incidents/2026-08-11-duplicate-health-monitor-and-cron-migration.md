---
type: incident
schema_version: 1
project: aiwebbuilder
component: ci / health-monitor
category: cost-and-infrastructure
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-11
resolved: 2026-08-11
verified: 2026-08-11
agents: [claude-code]
source_repo: StockHedge/aiwebbuilder
tags: [github-actions, cron, cloudflare-workers, monitoring, cost, duplicate-workflow]
---

# 죽은 저장소가 같은 프로덕션을 이중 감시하고 있었다

## 요약

계정 전체 GitHub Actions 무료 한도 소진(2026-08-11)을 조사하다가,
**`aiwebbuilder-contest` 저장소가 `aiwebbuilder`와 완전히 동일한 URL을 5분마다 이중 감시**
하고 있는 것을 발견했다. 월 215분(전체 한도의 **11%**)이 순수 낭비였다.

## 발견

두 저장소의 `health-monitor.yml`이 같은 대상을 본다.

```
aiwebbuilder          → aiwebbuilder-api.fly.dev/health, aiwebbuilder-main.pages.dev
aiwebbuilder-contest  → aiwebbuilder-api.fly.dev/health, aiwebbuilder-main.pages.dev  (동일)
```

`aiwebbuilder-contest`는 **2026-07-29 이후 커밋이 없다**. 마지막 3개 커밋이 전부
"듀얼 머신 마이그레이션" 정리 작업이다. 저장소 복사 시 워크플로가 함께 복제됐고,
텔레그램 알림도 같은 채팅방으로 중복 발송되고 있었다.

**조치**: `contest` 쪽 워크플로를 API로 비활성화했다(파일은 보존, 롤백 가능).
나머지 워크플로 3종(배포 2, 테스트 1)은 건드리지 않았다.

## 이관

`health-monitor.yml`을 Cloudflare Worker(`StockHedge/ops-cron`)로 옮겼다.

**부수 효과로 모니터링 품질이 올라간다.** GitHub은 개인 계정 스케줄 워크플로를 크게
throttle해서, `*/5`로 설정된 헬스체크가 **실측 45~80분 간격**으로 돌고 있었다
(하루 288회여야 하는데 18.5회). Worker는 설정대로 정직하게 돈다.

다만 그대로 옮기면 실행이 16배로 뛰므로 **`*/15`로 완화**했다. 실측 대비 여전히 3~5배
촘촘하고, 하루 576회(백엔드+프론트 2그룹) → 192회가 된다. 이 두 그룹은 embolos와 달리
백엔드에 기대주기 하드코딩이 없어 자유롭게 조정할 수 있었다.

## 이관하지 않은 것

`db-backup.yml`은 **Worker로 옮길 수 없다.** `pg_dump`로 Neon을 덤프해 R2에 업로드하는
실연산이라 Cloudflare Worker 런타임에서 불가능하다. Actions에 그대로 둔다(월 10분 수준).

> 초기 분류에서 이 워크플로를 "curl 전용"으로 오판했다. `grep -c curl`이 apt 키 다운로드
> 라인에 걸린 오탐이었다. **`checkout`/`setup-*` 유무로 판별해야 정확하다.**

배포 워크플로 2종(Fly, Cloudflare Pages)도 그대로 둔다.

## 교훈

저장소를 복사·포크할 때 **스케줄 워크플로가 함께 복제되면 대상이 겹친다.**
복사본은 배포·테스트만 남기고 모니터링·크론은 꺼야 한다. 이 항목은
[[ci-cron-trigger-billing-trap]] 의 "저장소별 예산이 계정 한도를 넘긴다" 절과 같은 뿌리다.

## 롤백

원본 워크플로는 삭제하지 않고 비활성화만 했다. 절차는 `ops-cron/docs/ROLLBACK.md`.
