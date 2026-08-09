---
type: milestone
schema_version: 1
project: for-marketing
completed: 2026-08-04
verified: 2026-08-04
agents: [claude-code]
source_repo: C:\Users\강지호\project\for-marketing
source_commit: 8d61a1b
tags: [marketing-automation, meta-graph-api, live-publish, cloudflare-r2, webhook]
---

# P7 실 Graph 발행 + R2 실연결 + Meta 웹훅 구독 — Live 전환 완료

## 무엇을 만들었나

- **Meta live 전환**: 60일 사용자 토큰을 무기한 페이지 토큰으로 교환. FB 페이지 "AI웹빌더"
  (Graph ID 1217995614733150), IG `official_aiwebbuilder` 연결. 토큰 매니저
  (store/refresh/health)를 신규 구현 — P1 명세에는 있었으나 실제로는 미구현이던 것을
  운영 부트스트랩 시점에 발견해 채웠다.
- **P7 실 Graph 발행·커뮤니티 클라이언트**: IG 3단 발행(컨테이너 생성 → status_code
  폴링 → media_publish), 댓글/DM 조회·응답을 stub에서 실구현으로 전량 교체. publish/
  community capability를 "페이지 토큰 존재 여부" 게이트에 연결해, 토큰이 없으면 여전히
  stub으로 안전하게 떨어지도록 유지.
- **R2 실연결**: 버킷 `marketing-hub-media`. 업로드 → 공개 URL GET 200 → 회수 → 동일
  URL GET 404까지 실왕복으로 확인.
- **Meta 웹훅 구독**: Graph API 조회로 active 확인 — instagram(comments, mentions),
  page(messages, messaging_postbacks, feed).
- **렌더링 에셋 저장소 편입**: Pretendard(OFL) 폰트 + 자체 합성 오디오 3종.
- **GH Actions cron 100분 주기로 통합**: 월 사용량 추정 ~3,800분 → ~700분(무료 한도
  2,000분 이내). 트레이드오프: 발행·승인 반영이 최대 100분 지연될 수 있음.

## 검증 근거 (실연동 — 코드 검토 아님)

- backend pytest 559 passed / ruff / mypy 클린. 프론트 typecheck 통과.
- 셋업 체크리스트 실프로브 10개 중 9 ok(embolos만 미설정 — 의도된 보류, 별도 트랙).
- R2: 업로드 후 공개 URL GET 200, 회수(삭제) 후 동일 URL GET 404 확인.
- Meta 웹훅 구독 상태를 Graph API로 직접 조회해 active 확인.

## 재사용 교훈

- [[s3-compatible-storage-silent-incompatibility]] — R2가 S3 관례인 객체별 ACL을
  지원하지 않아 "지연 비공개" 기능이 무동작이었던 것을 실왕복 테스트로 발견.
- [[gitignored-runtime-assets-break-deploy]] — 폰트·오디오를 gitignore했더니
  Dockerfile `COPY assets/`가 빈 디렉터리를 복사해 "내 기기에서만 되는 배포"가
  성립했던 것.

## 미해결

- `ig_user_id`가 Meta 측 전파 지연으로 아직 바인딩되지 않음 — 야간 잡이 매일 재시도.
- `check_container_published`: Graph API에 컨테이너 ID → media ID 역조회 필드가 없어
  `external_id=None`으로 폴백. 이론상 좁은 중복 게시 가능 구간으로 남아 있음(미해결로
  기록, 발생 시 재조사 필요).

## 관련

- 관련 인시던트: [[2026-08-04-secret-leaked-via-error-url]] — 같은 세션에서 부트스트랩
  오류 경로의 시크릿 유출을 발견·수정(별도 커밋 c66beef).
- 저장소 커밋: 8d61a1b(에셋), 15afe1e(P7), e131312(슬라이드쇼 수정), 98476d7(R2·웹훅),
  657c066(cron 100분 전환).
