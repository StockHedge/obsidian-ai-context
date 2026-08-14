---
type: incident
schema_version: 1
project: for-marketing
component: digest-pipeline
category: integration
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-11
resolved: 2026-08-12
verified: 2026-08-11
agents: [claude-code]
source_repo: StockHedge/for-marketing
source_commit: ce46ef2
tags: [실증, 권한, 외부api, 오진, meta, 첫완주]
---

# 파이프라인이 처음으로 완주했고, 마지막 한 구간이 권한 때문에 막혀 있었다

## 요약

[[2026-08-11-digest-pipeline-first-run-three-bugs]]의 수정분을 재실행해 **수신부터 승인까지
전 구간을 처음으로 완주**했다. 그 과정에서 결함 2건을 새로 발견했고, 그중 하나는
**7일간 "자동 해소되는 대기"로 잘못 분류돼 있던 항목의 진짜 원인**이었다.

## 실증된 것 (2026-08-11 15:17~16:15 KST)

| 구간 | 증거 |
|---|---|
| vibecoding → 허브 21건 | `허브 수신 완료 — 21건`, 식별자 경고 없음 |
| 출처명 저장 7개 커뮤니티 | `digest_batches` payload 실조회 — #18 해소 |
| 카드 10장 생성(24.7s) | `job_runs`에 11:42 실패와 15:18 성공이 나란히 남음 |
| 컴플라이언스 게이트 통과 | `compliance_violations: []` — #17 해소 |
| 캡션·표지·본문 출처 표기 | 카드 이미지 육안 확인 |
| Telegram 승인 카드 도착 | 사용자 수신 확인 |
| 승인 반영 | `approvals_log`: `approved` / `decision_source: telegram` |

발행만 남아 있었고, **2026-08-12 01:05 그것도 통과했다**(아래 '해결').

## 새로 드러난 결함

### #20 표지 pill이 마진을 넘은 채 그려졌다

경계 검사가 `_draw_pill` **뒤**에 있어 넘치는 pill 하나는 반드시 그려진 뒤 루프가 멈췄다.
주석은 "넘치면 생략"이라고 의도를 적어 뒀지만 코드는 "넘친 것까지 그리고 그 다음부터
생략"이었다.

실측이 판단을 뒤집었다: 긴 이름 두 개는 애초에 한 줄에 안 들어간다(486+16+467=969 >
가용 920). **첫 회차에 pill 2개가 보였다는 사실 자체가 침범의 증거였는데, 재는 것을
빼먹으면 그게 정상으로 보인다.** 커뮤니티 순서는 점수 정렬이라 회차마다 바뀌므로
잘림은 비결정적으로 나타났을 것이다.

→ 측정을 그리기에서 분리(`_pill_size`), 표지 pill은 사이트 단위로 축약,
생략분은 `+N`으로 노출. 캡션·본문 label은 원래 이름 유지(출처 책임은 그 두 곳이 진다).

### #21 `ig_user_id`는 전파 대기가 아니라 권한 부재였다 (7일 오진)

2026-08-04 부트스트랩에서 `instagram_business_account`가 안 와서 **"Graph 전파 지연"으로
판정**하고 매일 도는 잡에 자동 회수를 붙였다. `NOW.md`에도 "대기 중(자동 해소) — 손대지
말 것"으로 적혀 있었다. 7일간 회수되지 않았다.

단서는 잡 이력이었다 — **12시간 간격 두 실행(04:04, 16:31)의 결과가 완전히 동일**했다.
전파 지연이면 변화가 있어야 한다. 진단을 코드에 심고 한 번 돌리자 원인이 나왔다:

```
granted: ads_management, ads_read, business_management, instagram_content_publish,
         instagram_manage_comments, pages_messaging, pages_read_engagement,
         pages_show_list, public_profile
missing_for_ig: instagram_basic
```

**발행 권한은 있는데 조회 권한이 없다.** Graph는 권한 없는 필드를 오류가 아니라
응답에서 생략하므로, "아직 안 옴"과 "영원히 안 옴"이 구분되지 않았다.

같은 코드에 두 번째 결함이 겹쳐 있었다: `_collect_pages`가
`.values(ig_user_id=ig_user_id)`로 무조건 대입해, **값이 어쩌다 들어와도 다음 실행이
null로 지우는 구조**였다. 회수 잡이 자기가 회수한 값을 스스로 없앤다.
테스트로 실증했다 — 옛 로직에서 기존 값 `17841400000000000`이 `None`이 된다.

부수 발견: 같은 진단에서 `pages_manage_metadata` 부재로 **페이지 웹훅 구독도 매번
실패**하고 있었다(`webhook_subscribed: false`). `NOW.md`에는 "웹훅 구독 active"로
적혀 있었다 — 문서와 실물이 어긋나 있었다.

## 시도했지만 실패한 접근

`connected_instagram_account` 폴백을 추가해 배포·실행했으나 그 필드도 오지 않았다
(`ig_source: null`). **코드로 우회할 수 없다** — 권한 추가 또는 IG User ID 수동 입력이
필요하다. 이 실패 자체가 다음 세션이 같은 시도를 반복하지 않게 하는 근거다.

## 수정

| 대상 | 커밋 |
|---|---|
| `digest_card.py` — 그리기 전 경계 판정 + `+N` 표식 | `7048abd` |
| `digest_cards.py` — 표지 pill 사이트 단위 축약 | `7048abd` |
| `refresh.py` — `ig_user_id` 조건부 대입 + 미바인딩 진단 | `5348c4e` |
| `refresh.py` — `connected_instagram_account` 폴백 | `0572f2d` |

## 검증 결과

- pytest **619 passed**(회귀 테스트 6건 추가), ruff·mypy(113 files) 클린
- Fly **v17** 배포, 헬스 HTTP 200
- **회귀 테스트 유효성을 옛 로직 복원으로 확인** — #20에서 3건 중 2건 실패(1건은 옛
  로직에서도 옳은 케이스), #21에서 `assert None == '17841400000000000'`.
  → 패턴 승격: [[regression-test-must-fail-without-the-fix]]

## 해결 — 첫 IG 실게시 (2026-08-12 01:05)

permalink `instagram.com/p/Db5-jJcjSM3/` · media id `18337099675253565` (FEED/IMAGE).
Graph 조회로 실제 게시를 확인했다.

막고 있던 것은 **"instagram_basic은 제공되지 않는다"는 08-04 메모 한 줄**이었다.
실제로는 앱 유스케이스(Instagram API → 권한 및 기능)에 있고 버튼 두 번으로 추가된다.
Graph API **탐색기** 드롭다운에 없는 것을 "제공 안 됨"으로 단정한 오독이었다 —
탐색기는 *앱에 이미 추가된* 권한만 보여준다. 이 한 줄 때문에 7일간 권한 경로를
시도조차 하지 않았고, 그동안 원인을 "전파 지연"으로 오인했다.

권한 추가 후 토큰을 재발급하자 세 가지가 한 번에 풀렸다.

| 증상 | 해소 |
|---|---|
| `ig_user_id` 미바인딩 | `ig_source: instagram_business_account` — 정식 경로로 조회됨 |
| 웹훅 구독 403 | `pages_manage_metadata` 확보 → `webhook_subscribed: true` |
| 발행 `code=10` | `instagram_basic` 확보 → 컨테이너 생성·게시 성공 |

부수적으로 `dead_letter` 복구 경로가 없어 DB를 직접 만질 뻔했다.
`dead_letter → scheduled` 전이와 `POST /api/content/{id}/retry`(예산 리셋, 이력 보존)를
신설해 해결했다 — 자동 재시도가 아니라 **운영자가 원인 해소를 확인한 뒤 여는 경로**다.

## 다른 프로젝트에도 적용할 규칙

1. **권한 없는 필드는 침묵으로 온다** — [[silent-field-omission-vs-propagation-delay]]
2. **회귀 테스트는 되돌려 실패시켜 봐야 검증된다** — [[regression-test-must-fail-without-the-fix]]
3. **경계 검사는 부작용 앞에 둔다.** 그리고 나서 넘쳤는지 보면 이미 그려진 뒤다.
   파일 쓰기·외부 호출·상태 전이에도 같은 형태가 있다.
4. **부정형 사실은 근거와 확인 위치를 함께 적는다** —
   [[negative-claims-need-their-evidence]]. "없다/안 된다/지원 안 함"은 긍정형보다
   검증 비용이 크고 수명이 짧은데, 근거가 없으면 재확인할 방법조차 없다.
5. **종착 상태에도 되돌아올 문이 필요할 수 있다.** `dead_letter`는 콘텐츠가 잘못됐다는
   뜻이 아니라 재시도 예산이 소진됐다는 뜻이다. 외부 원인이 해소되면 같은 산출물을
   다시 쓸 수 있어야 하고, 그 문이 없으면 DB 직접 조작이 유일한 선택지가 된다.

## 관련

- 선행 사건: [[2026-08-11-digest-pipeline-first-run-three-bugs]]
- 결함 대장: for-marketing `docs/ai/HISTORY.md` #20·#21
