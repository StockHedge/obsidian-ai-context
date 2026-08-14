---
type: incident
schema_version: 1
project: for-marketing
component: digest-pipeline
category: integration
severity: high
status: resolved
root_cause_status: confirmed
discovered: 2026-08-11
resolved: 2026-08-11
verified: 2026-08-11
agents: [claude-code]
source_repo: StockHedge/for-marketing
source_commit: 6231bbf
tags: [운영장애, 계약불일치, llm출력, null방어, 첫실행]
---

# 카드뉴스 파이프라인 첫 실운영에서 버그 3건이 한꺼번에 드러났다

## 요약

vibecoding 다이제스트 → IG 카드뉴스 파이프라인을 배포하고 첫 회차를 돌리자
서로 무관한 결함 3건이 동시에 나타났다. 세 건 모두 **단위 테스트·타입체크·로컬
왕복 테스트를 전부 통과한 뒤** 실운영에서만 드러났다.

## 증상

1. 카드 생성 잡이 HTTP 500 — `LlmComplianceResult` ValidationError
2. 생성된 캡션의 출처가 비어 있음 — `· [] Claude Code의 기본…`
3. `poll_aiwb_daily`가 8회 연속 실패 (08-10 14:33 UTC부터, 획득 지표 롤업 중단)

## 증거와 재현 방법

- 잡 이력: `GET /api/jobs/recent` — 세 건 모두 운영 `job_runs`에 남아 있다
- 2번은 운영 DB 조회로 확정: `content_items` #1 draft, `media_asset_ids` 10장,
  캡션 앞부분에 빈 대괄호
- 3번 에러 원문: `NotNullViolationError: null value in column "utm_source"`,
  실패 행 `(97, 1, 2026-08-10, null, null, null, 1, 0, …)`

## 시도했지만 실패한 접근

로컬 왕복 테스트로 계약을 검증했다고 판단한 것이 오진이었다. 자세한 내용은 근본원인 2번.

## 근본원인

**세 건 모두 확인됨.**

1. **LLM 응답 스키마 과잉 제약** — `LlmComplianceResult.reasoning`이 필수였다.
   `tool_choice`로 스키마를 강제해도 모델이 필드를 생략할 수 있다. Haiku가 위반 없는
   캡션에 `{"passed": true, "violations": []}`만 반환했고, ValidationError가 나면서
   `submit_content` 전체가 예외로 끝났다. 판정의 본질은 `passed`/`violations`인데
   부가 정보 한 필드가 게이트 전체를 막았다.

2. **생산자 계약 오독** — `hub_sender.build_payload`가 `r.get("id")` / `r.get("name")`을
   읽었으나 `collect_community()`가 돌려주는 키는 `community_id` / `community_name`이다.
   `.get(k, "")`라 조용히 빈 문자열이 됐다. **놓친 이유가 본질이다**: 로컬 왕복 테스트의
   fixture를 내가 직접 `{"id", "name"}`으로 작성했다. 실물 왕복을 했다고 믿었지만
   실제로는 생산자 계약이 아니라 내 가정을 검증하고 있었다.
   → 패턴 승격: [[fixture-must-mirror-producer-contract]]

3. **`.get(k, default)`의 null 미방어** — 이 형태는 **키가 없을 때만** 기본값을 쓴다.
   `{"utm_source": null}`이 오면 `None`이 그대로 통과해 NOT NULL을 위반한다.
   같은 테이블에 쓰는 `rollup.py`는 처음부터 `.get(k) or ""`였다 —
   **두 경로 중 하나만 방어**하고 있었고, 둘 다 건드릴 일이 없어 드러나지 않았다.

## 수정

| 대상 | 커밋 |
|---|---|
| `llm_review.py` — `reasoning: str = ""` | `37a97d6` |
| `digest_cards.py` — 출처 없는 커뮤니티 제외 | `37a97d6` |
| `platform_sync.py` — `.get(k) or ""` | `6231bbf` |
| `hub_sender.py` — 키 수정 + 식별자 부재 경고 로그 | vibecoding `038be02` |

## 검증 결과

- pytest 612 passed(회귀 테스트 7건 추가), ruff·mypy(113 files) 클린
- **`poll_aiwb_daily` 실운영 수동 실행 성공** — `{"days": ["2026-08-10","2026-08-11"]}`.
  8회 연속 실패가 끊기고 밀린 이틀치가 롤업됐다
- 카드 경로 2건도 **08-11 15:17~16:15 재실행으로 실증 완료** — 수신 21건 → 카드 10장 →
  게이트 통과 → 승인 반영. 후속 사건: [[2026-08-11-digest-pipeline-first-complete-run]]

## 재발방지

- 회귀 테스트 7건: 세 버그 각각 + 출처 일부 누락 시 부분 처리
- 발신 측: 커뮤니티 식별자를 찾지 못하면 경고 로그 (조용한 빈 출처보다 시끄러운 편이 낫다)
- 수신 측: 출처를 표기할 수 없는 글은 카드로 만들지 않는다 (저작권 대응이 핵심이므로
  빈 대괄호로 발행하느니 빼는 쪽이 옳다)

## 다른 프로젝트에도 적용할 규칙

1. **LLM 응답 스키마에서 부가 필드는 필수로 두지 않는다.** 스키마 강제는 보장이 아니다.
   판정·결정에 쓰이는 필드만 필수로 하고, 설명·근거류는 기본값을 준다.
2. **`.get(k, default)`는 null을 막지 못한다.** 외부에서 오는 dict에는 `.get(k) or default`.
   같은 테이블·같은 자원에 쓰는 경로가 둘 이상이면 방어 수준을 맞춘다.
3. **fixture는 생산자가 실제로 만드는 구조를 복제해야 한다** — [[fixture-must-mirror-producer-contract]]

## 관련 커밋과 문서

- 허브: `37a97d6`, `6231bbf` (Fly v14)
- vibecoding: `038be02`
- 결함 대장: `docs/ai/HISTORY.md` #17~#19
- 선행 사건: [[2026-08-04-secret-leaked-via-error-url]] (같은 프로젝트, 계약 검증 부족 계열)
