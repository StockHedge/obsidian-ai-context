---
type: incident
schema_version: 1
project: game-factory
component: game-factory
status: resolved
root_cause_status: confirmed
게임: game-factory
분류: 배포·콘솔
심각도: 높음
발견일: 2026-07-23
해결일: 2026-07-24
상태: 해결
aliases: [AGY 헤드리스 세션 행, bare agy 좀비, 고아 서버 행]
태그: [agy-lane, headless, 프로세스관리, 파이프라인]
---

# AGY 헤드리스 세션이 작업 완료 후 종료하지 못하는 행 (좀비/고아 프로세스)

**대상**: crowd-clash(AGY) 풀오토 런 — dev·audit·autoship×2에서 총 4회 행, 수동 개입 6회.

## 증상
작업(커밋·빌드)은 완료됐는데 agy print 모드 세션이 종료하지 못하고 25분+ 유휴
(CPU 정지, cli.log에 6분 간격 keepalive만). 파이프라인 전체가 멈춤.

## 근본원인 (2종)
1. **고아 http.server** — 세션이 캡처용으로 띄운 8300번대 정적 서버를 안 끄고 종료 시도.
2. **bare 대화형 agy 좀비** — 세션이 터미널에서 인자 없이 `agy`를 실행 → 대화형 모드
   무한 입력 대기(한 런에서 최대 3개 발생). 자식 터미널이 살아 있으면 메인이 못 끝남.

## 수정
- 즉효 처치(수동 실증): 해당 좀비/고아 PID를 표적 종료하면 메인이 즉시 정상 마무리.
- **영구 수정(커밋 02bdaf7)**: AgyProvider 하트비트에 자동 정리 통합 — bare agy 좀비는
  파일 활동 정지 1분 내 즉시 제거, 8300번대 고아 서버는 6분 이상 정지 시 제거
  (사용 중 서버 오살 방지 가드).
- 헌법(.agents/AGENTS.md)에 프로세스 정리 의무 + bare agy 금지 명문화 — 단 행동 규칙만으론
  불충분함이 실증돼(audit·autoship에서 재발) 인프라 정리가 정답.

## 재발방지
- 잔류 프로세스 정리는 에이전트의 선의가 아니라 **하네스가 보장**한다.
- 워처의 quiet-N분 폴백이 행 감지 채널로 유효(상태 전환만 보면 장기 잡에서 오탐).
