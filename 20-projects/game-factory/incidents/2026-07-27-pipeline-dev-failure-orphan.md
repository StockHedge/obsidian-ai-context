---
게임: (파이프라인)
분류: 파이프라인
심각도: Critical
발견일: 2026-07-27
해결일: 2026-07-27
상태: 수정완료
aliases: [dev 실패 무전이, developing 고아, 게이트 미실행]
---

# 파이프라인 — dev 잡 실패가 상태 전이 없이 무기한 정지로 이어짐 (+ 동반 결함 4종)

18개 에이전트 병렬 진단 + 적대적 검증(전 항목 confidence high)으로 확인한 구조 결함들.
증상은 "color-gate(AGY)가 developing인 채 잡 큐가 비어 있다" 하나였지만, 원인은 서로 다른
5개 결함이었다.

## 증상

`color-gate(AGY)`가 2026-07-25 17:40~17:51에 dev→review→audit(3.86 PASS)→autoship을
완주했는데도 `game.json`의 status가 `developing`에 머물렀고, 대시보드 잡 큐는 비어 있었다.
`betaRounds:1`인데 `lastBeta` 필드는 아예 없었다.

## 근본원인

**타임라인 (git·잡 로그·베타 리포트 대조)**

1. 17:51 autoship 완료 → `approved`
2. 17:52 베타 실행, 총점 47.9 (합격선 70 미달)
3. `runner.py`의 베타 미달 분기가 `status="developing"` + `betaRounds=1`로 되돌리고
   dev 잡을 재투입 — **여기까지는 설계대로다**
4. 그 dev 잡(`9c53e3d530cf`)은 18:17에 실제로 작업을 마치고 커밋(`507140d`)까지 했다
5. 그런데 agy 프로세스가 종료되지 않아 45분 print-timeout으로 `Error: timeout waiting
   for response` → 잡은 **실패**로 판정
6. **dev 실패 분기가 알림만 보내고 `return` 했다** → 상태 전이·재시도·에스컬레이션 전무
   → `developing`인 채 대응 잡만 사라져 무기한 정지

**핵심 결함**: `_advance_pipeline`의 dev 분기만 종단 상태를 남기지 않았다. 다른 4개 잡
타입(review/improve/audit/autoship)은 전부 `_retry_or_escalate`를 타는데 dev만 예외였다.
정책 주석은 "첫 창작 세션은 자동 재시도 대상이 아니다"였는데, **재시도 금지**와 **상태
미기록**은 별개의 결정인데 하나로 뭉갠 구현이다.

**동반 결함 4종** (같은 진단에서 확인)

- **재시작 복구가 `betatesting` 한 상태만 처리**했다. 잡 큐는 메모리 전용인데
  전이 상태(developing/reviewing/auditing/improving/verifying)는 복구 대상이 아니어서
  서버가 죽으면 전부 고아가 된다.
- **자동 파이프라인이 정량 게이트를 한 번도 실행하지 않았다.** `gates.run_gates`는
  대시보드 수동 버튼에서만 호출됐다 — crowd-clash(AGY)가 `lastGateResult:null`인 채
  audit 4.43 합격을 받은 것이 증거다. 품질 구조 v1은 "게이트 통과 → review → audit"를
  요구하는데 자동 경로에 그 관문이 없었다.
- **잡 벽시계 한도가 파이썬 측에 전혀 없었다.** 엔진 CLI 자체의 타임아웃(agy 45분)에만
  의존해, CLI마저 응답하지 않으면 워커가 영구히 한 잡에 묶인다.
- **베타 점수를 합격 시에만 기록**했다. crowd-clash 실측 3회(28.6/26.5/31.3)와
  color-gate 47.9가 레지스트리에 한 줄도 남지 않았다 — 정작 진단에 필요한 것은 실패한
  회차의 점수와 리포트 runId다.

## 수정

`factory/runner.py`
```python
# 수정 전 — 알림만 보내고 return
if job.job_type == "dev":
    if not (job.result and job.result.success):
        await notify.send(...)
        return          # ← status 갱신 없음 = 고아

# 수정 후 — origin 별로 종단 상태를 반드시 남긴다
if job.job_type == "dev":
    if not (job.result and job.result.success):
        if job.origin == "kickoff":
            registry.update_meta(job.game_id, status="escalated",
                                 escalationReason="dev_failure")
            await notify.send(...)
        else:
            await self._retry_or_escalate(job, meta, title, reason)
        return
    if not await self._run_gates_or_escalate(job, title):
        return
```

- `Job.origin`(kickoff/develop/beta-redev) 신설 — 킥오프만 재시도 없이 즉시 escalated,
  베타/디벨롭 재투입은 지시서가 구체적이라 1회 재시도 대상.
- `_run_gates_or_escalate()` 신설 — dev/review/improve 직후 게이트를 `asyncio.to_thread`로
  실행(블로킹이므로 이벤트 루프 밖), 결과를 `lastGateResult`에 기록, 불합격이면
  `gate_failure`로 체인 중단.
- `JOB_TIMEOUT_S`(`FACTORY_JOB_TIMEOUT_MIN`, 기본 90분) 백스톱을 `asyncio.wait_for`로.
- 베타 결과를 합격·불합격 모두 `lastBeta`에 기록(`passed` 필드 포함).

`app.py` — lifespan 복구를 `TRANSIENT_STATUSES` 전체로 확장, 전이 상태는
`orphaned_restart`로 표면화. **적용 즉시 고아 2건이 드러났다**: color-gate(AGY)와
`mobile-claude-code-1`(2026-07-09부터 아무도 모른 채 verifying 고착).

`games.schema.md` — `escalationReason`에 `dev_failure`·`gate_failure`·`orphaned_restart`
추가, `lastBeta` 정의를 "합격 결과"에서 "합격·불합격 모두"로 정정.

## 재발방지

- **전이표 강제**: `_advance_pipeline`을 if-체인이 아니라 `(job_type, success) →
  (next_status, next_job)` 테이블로 재작성하고, 잡 타입 × {성공,실패} 전 조합이
  테이블에 존재하는지 검증하는 단위 테스트를 두는 것이 근본 처방이다(이번엔 미적용 —
  다음 라운드 과제).
- **"프로세스 메모리에만 존재하는 파이프라인 진행 상태를 만들지 않는다"**를 규약으로.
  신규 백그라운드 태스크는 디스크 마커 + lifespan 복구 훅과 한 쌍으로만 추가한다
  (`release_jobs.py`가 이미 `rel-*.running.json` 마커로 선례를 만들어 뒀다).
- 고아 감시: 전이 상태인데 대응 잡이 없는 게임을 주기적으로 검출해 알린다(미적용).

## 관련

- [[2026-07-27-webview-misdiagnosis]] — 같은 진단에서 확정된 오진
- [[2026-07-27-registry-forgery-stopgap]] — game.json 위조 방어
