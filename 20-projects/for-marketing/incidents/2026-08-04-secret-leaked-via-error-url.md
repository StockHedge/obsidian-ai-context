---
type: incident
schema_version: 1
project: for-marketing
component: backend/tokens, backend/orchestrator
category: secret-leak
severity: medium
status: resolved
root_cause_status: confirmed
discovered: 2026-08-04
resolved: 2026-08-04
verified: 2026-08-04
agents: [claude-code]
source_repo: C:\Users\강지호\project\for-marketing
source_commit: c66beef
tags: [secrets, meta-graph-api, error-handling, logging]
---

# Meta 토큰 부트스트랩 오류 문자열에 담긴 URL이 client_secret·토큰을 유출함

## 요약

Meta 토큰 부트스트랩이 Graph API 500으로 실패했을 때, httpx `HTTPStatusError`의 기본
문자열 표현에 담긴 **요청 URL 전체**(쿼리스트링에 `client_secret`과 토큰 포함)가 잡
오류 메시지로 그대로 저장·반환됐다.

## 증상

- 운영 환경에서 Meta 토큰 부트스트랩이 Graph 500으로 실패.
- 실패 사유 문자열에 시크릿이 담긴 쿼리스트링이 그대로 노출.
- 노출 경로 2곳: ① `job_runs.error` 컬럼(운영 DB에 영구 저장) ② 해당 잡을 조회한
  내부 cron API 응답의 `error` 필드.
- 노출 범위: 운영 DB 1행 + 이 세션의 대화 로그로 한정. 외부 인터넷 노출·제3자 열람
  없음.

## 증거와 재현 방법

- 수정 커밋 c66beef의 diff(`backend/app/tokens/refresh.py`)가 수정 전 동작을
  보여준다 — `refresh_tokens()`가 `_bootstrap()`의 예외를 정제 없이 그대로
  상위(`runner.run_job`)로 전파하고 있었다.

## 시도했지만 실패한 접근

해당 없음 — 발견과 동시에 원인을 특정해 수정했다.

## 근본원인

**확인됨.** 잡 핸들러 계층에는 오류 문자열 정제 함수 `_sanitize_http_error`가 이미
존재했고 다른 Meta 연동 경로에서는 쓰이고 있었지만, **토큰 부트스트랩 경로
(`refresh_tokens` → `_bootstrap`)에는 적용되지 않았다.** 정제 함수 자체의 결함이
아니라 "함수는 있는데 이 호출부만 누락"한 유형이며, httpx 예외의 `str()`가 요청
URL 전체를 담는다는 사실이 눈에 띄지 않아 놓치기 쉬웠다.

## 수정

1. `backend/app/tokens/refresh.py`: `refresh_tokens()`가 `_bootstrap()` 호출을
   try/except로 감싸 `_sanitize_http_error(exc)`로 정제한 사유만 알림
   (`record_alert`)과 반환값에 담도록 변경.
2. `backend/app/orchestrator/runner.py`: 잡 실행기(`run_job`) 자체에 URL
   쿼리스트링을 제거하는 **공통 스크러버**를 추가 — 개별 핸들러가 정제를
   빠뜨려도 `job_runs.error`와 API 응답에는 원문이 남지 않는 2중 방어.
3. 노출된 앱 시크릿 자체는 재발급하지 않기로 결정(사용자 판단: 노출 범위가 운영
   DB 1행과 이 세션의 대화로 한정되고 외부 노출이 없다는 근거).

## 검증 결과

커밋 c66beef 반영 후 backend pytest 스위트 그린(회귀 없음). 공통 스크러버가
URL의 쿼리스트링 부분만 치환하고 나머지 텍스트는 보존하는 것을 확인.

## 재발방지

- 외부 API 클라이언트 예외의 `str()`가 URL을 통째로 담을 수 있다는 전제를 잡
  실행기 레벨의 공통 방어로 처리했다 — 새 연동을 추가할 때마다 핸들러별 정제
  여부에 의존하지 않는다.
- 시크릿을 헤더가 아니라 쿼리스트링으로 전달하는 외부 API를 다룰 때 이 위험이
  상시 존재한다는 점을 같은 세션의 P7 작업(`meta_graph.py`)에서도 재확인해
  문서화했다.

## 다른 프로젝트에도 적용할 규칙

일반화 가치("오류 문자열은 유출 경로다 — 정제는 개별 핸들러가 아니라 공통
계층에서 한다")는 있으나, 이번 승격에서는 별도 패턴 노트를 만들지 않는다
(중복 방지, 승격 여부는 사용자 판단 대기).

## 관련 커밋과 문서

- 수정 커밋: c66beef (`backend/app/orchestrator/runner.py`,
  `backend/app/tokens/refresh.py`)
- 관련 마일스톤: [[2026-08-04-p7-live-publish-r2-webhook]]
