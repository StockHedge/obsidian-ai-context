---
type: milestone
schema_version: 1
project: embolos
status: monitoring
created: 2026-07-28
work_date: 2026-07-28
agents: [codex]
verification: local-test-and-browser
tags: [beta, acquisition, landing-page, seller-research, admin-review]
---

# Embolos — 핸드메이드·리빙 5인 베타 모집 기반

## 목적

공개 런칭 전, 상품과 사진을 이미 보유한 핸드메이드·리빙 셀러 5명에게서 독립몰 개설 경험을 검증할 수 있는 모집·운영 기반을 만들었다.

## 확인된 결정

- 베타는 실제 결제·실주문 없이 스토어 미리보기와 개설 경험만 검증한다.
- 대상은 인스타그램·스마트스토어·팝업 판매 경험이 있고 상품 3개 이상을 준비한 핸드메이드·리빙 셀러다.
- 4주 동안 주 1회 30분 1:1로 진행하며, 완주 보상은 Pro 3개월과 5만원 상품권이다.
- 보상은 긍정 후기나 사례 공개의 대가가 아니며 사례 공개는 선택 동의다.

## 구현 범위

- 플랫폼 `/`을 베타 랜딩으로 전환하고 `/beta` 신청, `/beta/guide` 준비표, 완료 화면을 추가했다.
- 신청은 계정·결제·테넌트·WorkOrder를 자동 생성하지 않고 기존 `ApprovalRequest(kind=beta_application)`의 사람 검토 대기열에만 저장한다.
- 운영자 `/admin/beta`에서 operator 이상이 후보를 선발·대기·미선발로만 처리하도록 했다. 상태 변경은 감사 로그를 남기며 외부 행동을 자동 실행하지 않는다.
- 베타 운영 스크립트·선발 기준·인터뷰 질문·피드백 기록 양식을 프로젝트 문서로 남겼다.

## 검증 근거

- `backend/tests/test_platform_beta.py`: 베타 랜딩의 정직한 카피, 준비표, 필수 동의, 성공 신청의 검토 큐 저장을 포함한 4개 테스트 통과.
- `ruff check app/storefront/platform.py tests/test_platform_beta.py` 통과.
- 로컬 브라우저에서 랜딩과 신청 폼을 데스크톱 및 390px 모바일로 확인했다.
- 운영자 인증이 필요한 `/admin/beta`의 실브라우저 권한 플로우와 실제 DB 저장은 아직 운영 환경에서 검증하지 않았다.

## 남은 확인

- 플랫폼 호스트와 테넌트 서브도메인에서 `/beta` 경계가 의도대로 강제되는지 보안 검토.
- 실제 테스트 DB에서 신청 저장·운영자 권한·상태 변경의 통합 검증.
- 배포, 실제 후보 모집, 이메일 연락은 수행하지 않았다.

## Figma 검수 (2026-07-28)

### 확인된 사실

- Figma 파일 `embolos 베타테스터 프로토타입`에는 데스크톱·390px 모바일 기준의 랜딩, 신청 폼, 신청 완료, 첫 가게 준비표 화면이 모두 있다.
- 타깃(핸드메이드·리빙), 실제 결제 없음, 4주 1:1, Pro 3개월·5만원 상품권, 상품 3개·무드·미리보기 링크 흐름과 참여 조건은 화면에 반영됐다.
- 모든 주요 CTA의 Figma `reactions`가 빈 배열이다. 즉 화면 설계는 있으나 Figma 프로토타입 클릭 전환은 아직 연결되지 않았다.

### 수정 우선

- 과장 표현을 베타 검증 범위에 맞게 바꾼다: `완벽히 검증`, `확실히 해낼`, `완벽하게 준비`, `선발 확률이 매우 높습니다`, `럭키 셀러`.
- `법적/운영 필수 체크리스트`, `실제 판매 전술`은 법률·실판매 지원을 이미 제공하는 것처럼 읽힐 수 있으므로 준비 항목·향후 확인 범위로 낮춘다.
- 3일 이내 문자/이메일 안내는 실제 운영 SLA가 확정된 경우에만 유지한다.

### 다음 검증

- 랜딩 → 신청 폼 → 완료, 랜딩 → 준비표 → 신청 폼, 완료 → 준비표/랜딩의 CTA 전환을 Figma prototype reactions로 연결한 뒤 클릭 흐름을 재검수한다.

## 베타 운영 스프레드시트 (2026-07-28)

- 네이티브 Google Sheets 생성 확인: [엠보로스 베타테스터 운영판](https://docs.google.com/spreadsheets/d/1A6voBe6MIcL1daU6kh2PnsDPkCKGzwcbt_Ue2s65uL4/edit)
- 대시보드, 후보 모집, 선발 인터뷰, 4주 테스트, 피드백 이슈, 보상·동의, 주간 리뷰, 코드북의 8개 탭을 포함한다.
- 후보·인터뷰·테스트·이슈·보상·리뷰 탭은 Google Sheets 네이티브 표와 상태 드롭다운으로 변환했다.
- 가져오기 응답의 native conversion, 탭 메타데이터, 대시보드 수식, 후보·주차 테이블의 핵심 헤더를 재조회해 확인했다. Google 웹 UI의 육안 검수는 별도로 수행하지 않았다.

## 빠른 참조

- 운영 키트: `C:\Users\jihon\projects\embolos\docs\marketing\2026-07-베타-운영-키트.md`
- Figma 브리프: `C:\Users\jihon\projects\embolos\docs\marketing\figma-beta-landing-brief.md`
- Claude Code 인계: `C:\Users\jihon\projects\embolos\docs\marketing\claude-code-beta-handoff.md`
- 구현: `backend/app/storefront/platform.py`, `backend/app/admin/control_routes.py`
