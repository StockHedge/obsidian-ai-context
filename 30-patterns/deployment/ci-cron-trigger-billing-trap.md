---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-08-11
updated: 2026-08-11
source_projects: [embolos, for-marketing, aiwebbuilder]
agents: [claude-code]
tags: [github-actions, cron, billing, cloudflare-workers, cost, secrets-rotation]
---

# CI 러너를 크론 트리거로 쓰면 과금이 실연산의 6배가 된다

## 문제

GitHub Actions 무료 한도 2,000분/월을 **10.5일 만에 100% 소진**했다(2026-08-11).
저장소를 늘린 것도, 무거운 작업을 추가한 것도 아니었다.

원인은 연산량이 아니라 **과금 단위**였다.

- GitHub Actions는 run이 아니라 **job 단위**로 과금한다.
- job마다 소요시간을 **분 단위로 올림**한다. 8초짜리 job도 1분이다.
- skipped job은 과금되지 않는다.

`embolos/scheduled-jobs`가 전형이었다.

```
job 11개 중 9개 skipped, 실제 실행은 2개
실 연산: ops-watchers 13초 + frequent 8초 = 21초
과금:   2 job × 1분 올림       = 2분
```

**21초 일하고 2분을 냈다.** 375회 반복해 562분. 실 연산 대비 효율 17%.
계정 전체로는 wall-clock 1,247분에 과금 1,968분 — **721분(37%)이 순수 반올림**.

문제 워크플로의 공통점은 하나였다. **러너에서 아무 연산도 하지 않는다.**
ubuntu 러너를 부팅해 `curl -X POST` 한 줄 던지고 끝. 전체 소비의 66%가 이 형태였다.

## 재사용 가능한 원칙

1. **CI 러너는 컴퓨트 실행기이지 크론 스케줄러가 아니다.**
   워크플로가 `checkout` 없이 `curl`만 한다면 그건 크론 트리거이지 CI가 아니다.
   트리거는 트리거 전용 인프라(Cloudflare Workers Cron, 외부 크론 서비스)로 보낸다.

2. **비용은 run 수가 아니라 `job 수 × 분 올림`으로 계산한다.**
   "15분마다 도니까 월 96회 × 20초 = 32분"은 틀렸다. job이 2개면 96 × 2 = 192분이다.
   워크플로 하나를 job 여러 개로 쪼개는 순간 비용이 job 수만큼 곱해진다.

3. **job을 쪼개는 이유가 "가독성"이면 비용을 확인하라.**
   `if: github.event.schedule == '...'` 로 job을 분기하는 패턴은 읽기 좋지만,
   같은 시각에 발화하는 job이 2개면 매번 2분이다. 단일 job + 내부 분기가 싸다.

4. **저장소별로 예산을 세우면 계정 한도를 넘긴다.**
   for-marketing은 주석에 "무료 2,000분/월 內 운영이 목표(월 ~700분 추정)"라고
   적어두고 실제로 지켰다(월 476분). embolos도 자기 기준으로는 합리적이었다.
   **한도는 계정 단위인데 예산은 저장소 단위로 세운 것**이 구조적 결함이었다.

5. **스케줄 빈도를 낮춰도 기대만큼 안 줄어들 수 있다.**
   GitHub은 개인 계정의 스케줄 워크플로를 크게 throttle한다. `*/5`로 설정된
   헬스체크가 실측 45~80분 간격으로 돌고 있었다(하루 288회여야 하는데 18회).
   **binding constraint가 크론이 아니라 throttle**이면 빈도 하향은 무효다.
   절감은 "제거·이전·job 통합"처럼 결정적인 수단으로 해야 한다.

## 적용 조건

- 워크플로가 `actions/checkout`·`setup-*` 없이 HTTP 호출만 한다
- 스케줄 트리거가 주된 발화 경로다
- 한 워크플로가 `if:` 로 분기된 job을 여러 개 갖고 있다
- 여러 저장소가 하나의 CI 계정 한도를 공유한다

## 적용하지 말아야 할 조건

- 러너에서 실연산이 필요한 것은 옮기지 않는다.
  pytest 실행, 빌드, `pg_dump` 같은 작업은 Worker에서 불가능하다.
- 크론이 몇 개 안 되고 job도 1개라면 이관 비용이 절감보다 크다.
- 배포 워크플로(`deploy`)는 CI에 두는 편이 추적성 면에서 낫다.

## 확인 절차

```bash
# 저장소별 실행 수 (과금 아님 — 규모 파악용)
gh api "repos/OWNER/REPO/actions/runs?created=%3E%3D2026-08-01&per_page=1" --jq '.total_count'

# 실제 과금 추정: run 하나의 job 목록에서 skipped 제외하고 분 단위 올림
gh api "repos/OWNER/REPO/actions/runs/RUN_ID/jobs" \
  --jq '.jobs[] | select(.conclusion != "skipped") | "\(.name) \(.started_at) \(.completed_at)"'
```

**`/timing` 엔드포인트는 신뢰하지 말 것.** `billable.*.total_ms`가 0을 반환하는
경우가 있다(2026-08-11 실측). job 목록의 `started_at`/`completed_at`으로 직접 계산한다.

워크플로가 트리거 전용인지 판별하려면:

```bash
# checkout/setup 이 0이고 curl 이 있으면 이관 후보
grep -cE "uses: actions/(checkout|setup-)" .github/workflows/X.yml
grep -c "curl" .github/workflows/X.yml
```

## 함께 배운 것 — 시크릿을 .env에 안 두면 로테이션 외에 길이 없다

이관 과정에서 `embolos`의 `CRON_SECRET`·`BILLING_CRON_SECRET` 값을 어디서도
복구할 수 없었다. Fly secrets도 GitHub Actions Secrets도 **write-only**라 조회가
안 되고, 로컬 `backend/.env`에는 이 두 키만 빠져 있었다. PowerShell 히스토리에도 없었다.

결과적으로 **머니패스 자격증명을 강제 로테이션**해야 했다(Fly → GitHub → Cloudflare
순서로 심어 403 창을 최소화). 다른 키는 전부 `.env`에 있었기에 이 두 개만의 문제였다.

교훈: **외부에서 주입하는 시크릿도 로컬 `.env`(gitignore 적용)에 사본을 둔다.**
"어차피 Fly에 있으니까"는 조회 불가라 사본이 아니다.

관련: [[secrets-oauth-ops]], [[ps-korean-safe-write]]

## 근거 사건

- `20-projects/embolos/incidents/2026-08-11-actions-quota-exhausted.md`
- `20-projects/for-marketing/incidents/2026-08-11-actions-quota-exhausted.md`
- `20-projects/aiwebbuilder/incidents/2026-08-11-actions-quota-exhausted.md`
- 이관 결과물: `StockHedge/ops-cron` (Cloudflare Worker), `docs/ROLLBACK.md`에
  2026-09-01 재검토 기준 수록
