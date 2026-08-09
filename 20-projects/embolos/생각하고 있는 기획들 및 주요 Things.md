1. 49,000원 ~ 99,000원에 셀러가 원하는 디자인대로 페이지를 구조해줌. (외주기능)
2. 엠보로스 이전 센터로, 이전 직전 사전 신청 및 우리 플랫폼 프로 플랜 이상급으로 3달 이상 계약하면 해당 
3. 고객센터 자동화 시스템 구축
4. 통합 콘솔 시스템 구축 (**Embolos/embolos 종합 컨텍스트 마이그레이션.pdf**에 존재함)
5. 고객 쇼핑몰 APP을 만들어 주는 파이프라인 구축해야 함.
6. 고객 호스팅 페이지를 유료 과금을 통해 호스팅 플랜을 높혀주는 거 (후불)
7. AI Data 셀러/컨슈머 애널리시트 페이지 구축
8. 


---

## 제어면 구현 링크 (2026-07-26)

4번 통합 콘솔 = repo [`docs/admin_control_plane.md`](file:///C:/Users/jihon/projects/embolos/docs/admin_control_plane.md) + `/admin` 제어면.

- WorkOrder 타입: design_service / market_import / app_build / plan_upsell / ai_analytics
- 2번 마켓 이전 = `market_import` (일회성 ETL, 양방향 sync 금지)
- 3번 CS = Inbox/Intel
- PDF「종합 컨텍스트 마이그레이션」= HITL·Stage1~2·제어층 원칙


## 제품면 마감 (#4 제외, 2026-07-26)

- #4 = AI company **별도 트랙**
- 허브 `/seller/services` · `docs/things_product_surface.md`
- #1 design-service · #2 migrate+Pro3 · #3 CS알림 · #5 app · #6 업셀 CTA · #7 AI/소비자
- test 배포 반영


## #4 AI Company Ops (2026-07-26 구현)

- 헌법: `docs/오케스트레이션_논박_및_재구조안.md`
- 경계: `docs/ai_company_ops.md`
- 앱: `embolos-ops` (`fly.ops.toml`, `app.ops.main:app`)
- W2/W3/W4/W6 + digest · W1: `docs/ops_w1_health_ping.md`
