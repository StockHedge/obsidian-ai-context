---
type: pattern
schema_version: 1
status: active
verified_projects: [for-marketing]
candidate_projects: [embolos, aiwebbuilder]
created: 2026-08-04
agents: [claude-code]
tags: [pytest, secrets, test-isolation, side-effects]
---

# 패턴: 테스트는 외부 자격증명을 빈 값에서 시작한다

## 문제

로컬 `.env`에 실제 API 키·봇 토큰을 두는 개발 관례에서, 테스트 프레임워크가 설정 로더를
통해 그 값을 상속하면 **테스트가 실제 외부 부작용**(메시지 발송, 과금 API 호출, prod DB
접속)을 일으킨다. DB 격리만으로는 막을 수 없고, 모킹을 "깜빡한" 테스트 하나면 충분하다.

## 해법 (for-marketing에서 검증)

1. conftest에 **autouse 픽스처**로 외부 자격증명 환경변수 전체 목록을 빈 문자열로 강제
   + 설정 싱글턴 캐시 초기화. 새 외부 연동을 추가할 때 이 목록에 키를 함께 등록한다.
2. 특정 값이 필요한 테스트만 명시적 주입 헬퍼(`settings_env(...)`)로 그 테스트 스코프에서 재설정.
3. 모킹은 클래스 레벨 패치가 아니라 **통합 지점 함수**를 패치한다
   (예: httpx.AsyncClient.post를 패치하면 ASGI 테스트 클라이언트 자신까지 가로챈다 — 실측 함정).

## 판별 신호

- 테스트가 갑자기 건당 수 초씩 느려짐(네트워크 호출)
- 실제 채널(Telegram·메일함)에 테스트 산출물 도착
- CI(자격증명 없음)에선 통과하는데 로컬에서만 다른 결과

## 관련

- 검증 사건: [[2026-08-03-tests-called-real-external-apis]] (for-marketing, resolved)
- embolos의 "pytest는 test-branch.env 소싱 필수" 규약은 같은 문제의 DB 버전 —
  [[2026-08-01-pytest-prod-db-near-miss]] 참조
