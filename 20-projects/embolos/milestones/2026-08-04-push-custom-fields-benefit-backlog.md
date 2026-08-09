---
type: milestone
schema_version: 1
project: embolos
status: completed
created: 2026-08-04
work_date: 2026-08-04
agents: [claude-code]
verification: adversarial-review-3rounds(16-findings-fixed) + anchors(custom-fields-77, dispatch-53, coupon-zone-10, characterization-55, isolation-x2) + ci-green + tsc-clean
source_repo: StockHedge/embolos + StockHedge/embolos-app
source_commit: d242049→974dcf3 (backend) · 94631ba→67471e1 (app) · prod v37
tags: [push-triggers, custom-fields, coupon-zone-curation, d13-parity, prod-deploy, adversarial-review]
---

# Embolos — prod v37 + 푸시 후속 + 커스텀 필드 0065 + 혜택 백로그 D5·D13

AI 분석 트랙(별도 노트)에 이어 같은 날 완결된 4묶음. 사용자 지정 순서 A+B→C→D.

## 무엇이 끝났나

1. **prod v37 배포** — 0075~0078 마이그레이션을 release 로그로 실증(리비전 라인 확인이
   검증 경로 — prod DB 직접 조회는 auto classifier가 차단) + 스모크. AI트랙·푸시 인프라 라이브
   (cron 발화는 main 병합 대기 — dormant).
2. **A. 푸시 후속**(`d242049`·앱 `94631ba`) — 예약 셀러 push(notify_new_booking 게이트)·재입고
   buyer push(원자 클레임 집합 상속)·앱 수신 토글(옵트아웃 플래그, 서버 해제 성공 시에만
   로컬 전이).
3. **C. 커스텀 필드 0065**(`cd7bcda`+0079) — E3 원안(4스코프 1엔진 편집 화면) 실현: 콘솔
   CRUD(조건부 표시·레거시 가져오기)·체크아웃 배선(스냅샷 병합=기존 표면 무변경 합류)·가입
   배선·`/account/profile` 신설. 0079=값 유니크에 scope 편입.
4. **D. 혜택 백로그**(`7ce7450`·`974dcf3`·앱 `67471e1`) — D5 쿠폰존 수동 큐레이션 개통
   (0080: 핀·순서 축, amount 정렬 기각) + D13 개통(카드 상태 축 additive·앱 사다리·찜 API
   뱃지). serial/coupon kind는 머니패스 결정 2건을 승인 게이트로 명시한 착수 계획서로 고정
   (`docs/ai/plans/serial-coupon-kind-plan.md`).

## 재사용 가능한 교훈

- **커밋 전 적대 리뷰(렌즈별 발견→반증 검증)가 상시 실결함을 잡는다**: 3회전에서 확정 16건
  (blocker 1: Jinja 리스트 컴프리헨션 — 화면 전체 500 / high: UOW INSERT-before-DELETE 유니크
  500, 체크박스 해제 미저장 / medium: 야간 무조건 보류·flush 재발송 문구 전락·설정 변경이
  진행 중 결제를 깨는 미지 key 하드 오류 등). 반증 단계 기각 0 — 렌즈 분리가 유효.
- **Jinja 템플릿은 컴파일 게이트 테스트를 함께 커밋**: 콘솔 테스트가 HTTP를 못 도는 구조
  (셀러 세션=Redis)에서 문법 오류는 컴파일 테스트만이 잡는다.
- **"미지 key=사전 제거" 관례**: 셀러 설정 변경(필드 삭제·rename·토글)이 진행 중인 구매자
  제출을 깨지 않으려면 검증은 비활성 포함 로드 + raw 사전 필터로. 엔진 docstring 계약을
  배선 계층이 무효화하지 않는지 리뷰 렌즈로 확인할 것.
- **SQLAlchemy UOW는 같은 테이블 INSERT를 DELETE보다 먼저 낸다** — 교체 저장(delete→insert)
  은 delete 후 flush 1회가 필수(유니크 위반 500).
- **additive 파리티 축의 계약**: optional 필드 + "null=미판정(클라이언트 재계산 금지)"를
  타입 주석으로 명문화하면 교차 테넌트 표면(홈/검색/통합 찜)의 의도된 미판정이 안전하게 공존.

## 남은 것

실기기 검증(푸시 수신+신규 화면— 마일스톤①) · main 병합(cron 발화) · 다음 배포에 0079·0080
동반 · serial/coupon 결정 2건. 정본은 repo `docs/ai/NOW.md`.
