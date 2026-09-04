---
type: session-capture
schema_version: 1
status: promoted  # 2026-08-03 사용자 위임 검토 — 사건·패턴·마일스톤 3건 승격(하단 참조)
project: embolos
agent: claude-code
created: 2026-08-01
review_by: 2026-08-08
contains_verified_result: true
tags: [benefit-track, public-surface, incident-candidate, pytest-prod-guard]
---

# embolos T3 공개면 노출 완결 + pytest→prod 실사고·구조적 가드

## 세션 요약 (검증됨)

혜택 트랙 T3(공개면 노출) A·B·C 3트랙 완결 — 커밋 8개(`50cccb6`~`e0c53fb`) push.
표시 금액 = 청구 금액 계약이 [뱃지 → 체크아웃 라인·넛지 → 결제·완료 라벨 → 셀러 상세 →
이메일 영수증] 전 동선으로 확장. 신규 테스트 ~60건 + 앵커 3종 게이트, 브라우저 실측
(라이트/다크/vivid 뱃지·품절 사다리·PDP notice·쿠폰존 카드·`?code=` 프리필 중첩 2라인) 통과.
상세는 저장소 `docs/ai/NOW.md`·`docs/ai/plans/T3-public-surface.md`.

## 승격 후보 ① — 사건: pytest가 prod DB로 향함 (데이터 손실 위험 조건 해당)

- **경위**: `ruff check && SP=…; . "$SP/test-branch.env"` 체인에서 ruff 실패(F811)로
  `&&` 단락 → `SP` 빈 문자열 → env 소싱이 조용히 생략 → pytest가 로컬 `.env`(=prod DB)로
  실행. 시드 테넌트가 prod에 INSERT됐다가 teardown DELETE로 정리.
- **실측 정합**: prod READ ONLY 점검 — 시드 잔존 0건, alembic `0060` 불변. 오염 없음.
  prod 리비전이 뒤처져(`products.notice_category` 부재) 즉시 에러로 드러났지만,
  스키마가 같았다면 침묵 진행했을 사고.
- **구조적 차단(커밋 `88c9d4f`)**: `tests/conftest.py` 최상단에서 `PYTEST_DB_OK`
  (scratchpad `test-branch.env`에만 실리는 마커) 부재 시 `SystemExit`. 환경 규율(주의)이
  아니라 구조로 차단.
- **재사용 교훈(30-patterns 후보)**: ① env 소싱은 명령 체인 **맨 앞** + `|| exit`,
  린트 등과 `&&`로 묶지 말 것. ② "위험한 기본값 + opt-in 마커" 패턴 — 안전한 환경이
  자신을 증명해야 실행이 열린다. ③ 시크릿 마스킹 sed는 접미 변형(`URL_DIRECT=`)까지
  커버(`(PASSWORD|SECRET|KEY|URL[A-Z_]*)=`) — 이번에 테스트 브랜치 자격증명이 채팅에
  1회 노출된 원인(prod 아님, git·문서 기록 없음).

## 승격 후보 ② — 마일스톤: 혜택 트랙 T1·T2·T3 전체 완결

죽은 0064 엔진(런타임 소비 0) → 빌더(T1) → 구세대 3세계 이관·은퇴(T2) → 공개면 노출(T3)
3단계가 모두 완결. 핵심 설계 자산: D7(뱃지=엔진 실호출 승자) · D8(거절 allowlist
fail-closed) · D9(반사실 검증 — 사유별 단독 완화) · D12(주문 전 라이브/주문 후 원장
스냅샷 이원) · D14(코드 진열대 금액 슬롯 금지). "표시 후보도 청구 계산기와 같은 함수를
지난다"는 원칙은 타 프로젝트의 가격 표시 표면에도 재사용 가능.

## 처리 결과 (2026-08-03 — 사용자 위임 검토)

- 승격 ①: [[2026-08-01-pytest-prod-db-near-miss]] (incidents) + [[env-sourcing-optin-guard]] (30-patterns/ai-reliability)
- 승격 ②: [[2026-08-03-benefit-track-t1-t3-complete]] (milestones)
- cursor Inbox 2건(7/27): 정본(SYNC-STATUS·NOW.md) 중복·보고성 세션로그 — 90-archive 보관.

## 미해결

- prod 실배포·main 병합·pytest CI 편입은 사용자 결정 대기(NOW.md).
- 사용자 질문 접수(2026-08-01): 플랫폼 색감 밝은 시안 — 과거 요청 이력 없음 확인, 착수 대기.
