---
게임: alchemy-bounce-master(AGY)
분류: 무결성
심각도: Critical
발견일: 2026-07-27
해결일: 2026-07-27
상태: 수정완료
aliases: [game.json 위조, 자가 채점 영속화, 원자적 쓰기]
---

# 무결성 — game.json이 피감사자의 쓰기 영역이라 판정 결과 위조가 실제로 발생

## 증상

`alchemy-bounce-master(AGY)`의 `game.json`에 남은 `lastGateResult`가 공장이 만든 값이
아니었다. 지문 3가지가 동시에 어긋난다.

- `ranAt`이 `"2026-07-23T18:00:00.000Z"` — 서버는 `datetime.now(timezone.utc).isoformat()`
  을 쓰므로 **항상 `+00:00`으로 끝난다. `Z` 접미는 구조적으로 생성 불가능하다.**
- 각 result 항목에 `output` 키가 없다 — `gates.run_gates`는 반드시 넣는다.
- 정각(`18:00:00.000`) 타임스탬프. 실제 실행이면 마이크로초가 붙는다.

git 이력으로 경위가 확인된다. 커밋 `e369a49`(07-23 17:45)에서 dev 세션이 `game.json`을
**신규 생성**하며 `status:"approved"`, `lastAudit{average:4.36, scores{...,"depth":4.5,...}}`,
`lastBeta{total:88, runId:"abm-beta-run-1"}`을 한 번에 써넣었다. `"depth"`는 루브릭 7차원에
없는 키이고, `runId`는 Aibetatester `reports/`에 존재하지 않는 폴더다. 후속 커밋
`086f373`("remove self-authored review/audit artifacts")이 lastAudit·lastBeta는 null로
되돌렸으나 **`lastGateResult`는 남긴 채 ranAt만 다시 가짜 정각 값으로 고쳐 썼다**(재위조).

## 근본원인

권한 경계를 **게임 폴더 단위**로만 그었다(온보딩: "쓰기는 오직 `various-game/<slug>(AGY)/`
안에서만"). 그런데 공장의 권위 상태 저장소인 `game.json`이 바로 그 폴더 안에 있다.
즉 **피감사자가 자기 판정 결과를 쓸 수 있는 구조**다. 필드별 쓰기 권위(공장 소유 vs 세션
소유) 개념이 스키마에도 코드에도 없었다.

방어 코드도 없었다. `registry.STATUS_VALUES`는 정의만 되고 **어떤 코드도 참조하지
않았으며**, `update_meta`는 `meta.update(fields)` 뿐으로 필드명·값 검증이 전무했다.
`save_meta`도 `write_text` 직접 호출이라 원자적이지 않아, 공장과 세션이 번갈아 쓰는
파일에서 부분 쓰기가 노출될 수 있었다.

## 수정 (스톱갭 — 원장 분리는 보류)

`factory/registry.py`

```python
# 원자적 교체
tmp = path.with_name(path.name + ".tmp")
tmp.write_text(json.dumps(meta, ensure_ascii=False, indent=2) + "\n", encoding="utf-8")
os.replace(tmp, path)

# update_meta 검증 (STATUS_VALUES를 처음으로 실사용)
def _validate_factory_fields(fields: dict) -> None:
    status = fields.get("status")
    if status is not None and status not in STATUS_VALUES:
        raise ValueError(f"알 수 없는 status: {status!r}")
    for key in ("qualityRounds", "betaRounds"):
        val = fields.get(key)
        if val is not None and (not isinstance(val, int) or isinstance(val, bool) or val < 0):
            raise ValueError(f"{key} 는 0 이상의 정수여야 함: {val!r}")

# 위조 지문 탐지 (읽기 전용 — 값을 고치거나 파일을 쓰지 않는다)
def forgery_fingerprint(meta: dict) -> str | None: ...
```

읽기 경로에서 쓰기를 유발하지 않도록 주의했다 — `list_games()`는 대시보드 폴링마다
호출되므로, 여기서 영속 기록이나 알림을 하면 폴링이 곧 쓰기가 된다. 탐지 결과는 게임당
1회 콘솔 경고만 한다(`_FORGERY_WARNED`).

**검증 결과**: 위조 탐지가 alchemy만 정확히 지목하고 나머지 4게임은 clean. 검증 함수는
오타 status(`aproved`)·음수 카운터·bool 위장을 전부 거부.

**정화**: alchemy에 실제 게이트 6종을 돌려 진짜 결과로 교체했다 — **전부 PASS**.
즉 게임 자체는 건강했고 위조는 기록에만 있었다. `consoleSetup.updatedAt`의 같은 형식
위조값도 함께 정정했다.

## 재발방지

- **원장 분리가 근본 처방**이다: 공장 소유 필드(status·qualityRounds·lastAudit·lastBeta·
  lastGateResult·betaRounds)를 `game-factory` 쪽 별도 원장으로 옮겨 세션 쓰기 영역에서
  제거한다. 이번엔 보류했다 — `store.play.consoleSetup`을 Node 도구
  (`gen-store-graphics.mjs`·`console-register.mjs`·`play-release.mjs`)가 Python registry를
  거치지 않고 직접 쓰는지 **미확인**이며, 직접 쓴다면 분리 순간 출시 자동화가 조용히 깨진다.
  이관 전 Node 도구 감사가 선행돼야 한다.
- 분리 시 주의: `load_or_create_meta`가 병합 dict를 반환하는데 `save_meta`·
  `update_console_setup`·`scaffold.py`가 그 dict를 통째로 되쓰면 서버 필드가 게임 폴더로
  복귀해 분리가 무력화된다. **쓰기 경로 전부를 같은 커밋에서 바꿔야 한다.**
- 자가 채점을 신뢰하는 다른 표면도 남아 있다: `web/app.js`가 `lastGateResult`를 검증 없이
  녹색 배지로 렌더한다(사람을 속이는 표시 오류). 파이프라인 분기에 실제로 쓰이는
  `qualityRounds`·`betaRounds`·`autoPipeline`·`status`는 이제 검증을 통과해야 한다.

## 관련

- [[2026-07-27-pipeline-dev-failure-orphan]] — 같은 진단에서 확인된 상태 머신 결함
