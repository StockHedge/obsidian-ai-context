---
type: incident
schema_version: 1
project: game-factory
component: environment
status: resolved
root_cause_status: confirmed
게임: 공통
분류: 환경설정
심각도: 보통
발견일: 2026-07
해결일: 2026-07
상태: 해결
aliases: [로컬 포트 충돌 캐시 혼선, 게임별 포트 분리]
태그: [dev-server, 브라우저캐시, 포트]
---

# 로컬 테스트 포트 공유 → 브라우저 캐시가 게임을 뒤섞음

## 증상
여러 게임을 같은 localhost 포트로 로컬 테스트할 때, **브라우저 캐시가 이전 게임 자산을
재사용**해 다른 게임이 섞여 로드됨(잘못된 화면 / 동작).

## 근본원인
동일 origin(같은 포트) 재사용 → 캐시 / 서비스워커 / 스토리지가 게임 간 공유됨.

## 수정
게임별 **포트 분리**: 8080 = order-pop, 8123 = coop TD (게임마다 고유 포트 고정).

## 재발방지
- 게임별 dev 포트를 표준으로 고정하고 문서화.
- 필요 시 하드 리로드 / 캐시 무효화 병행.

## 관련
- 공용 패턴: [[isolate-local-origins-by-project]]
- 과거 비공유 식별자: `local-test-ports` (역사 기록, 기준 정보 아님)
