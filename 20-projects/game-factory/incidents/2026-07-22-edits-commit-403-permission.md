---
type: incident
schema_version: 1
project: game-factory
component: game-factory
status: resolved
root_cause_status: confirmed
게임: game-factory
분류: 프로세스오진
심각도: 높음
발견일: 2026-07-22
해결일: 2026-07-22
상태: 해결
aliases: [edits.commit 403, 서비스계정 앱초안 권한 누락]
태그: [play-api, service-account, 권한, 오진]
---

# Play Publishing API `edits.commit` 403 = 서비스계정 권한 누락 (설문순서 오진)

**대상**: `game-factory/tools/play-release.mjs` (신규 draft 앱 첫 커밋)

## 증상
신규 draft 앱에 대한 `edits.commit` 호출이 **403**.
초기엔 "설문(앱 콘텐츠) 작성 순서가 틀려서"라고 **오진**.

## 근본원인
서비스계정에 **"앱 초안 생성·수정·삭제(create/edit/delete drafts)" 권한**이 없었다.
설문 순서와 무관. Play Console 사용자·권한에서 서비스계정 역할 부여 문제.

## 수정
Play Console → 사용자 및 권한 → 해당 서비스계정에 **앱 초안 관리 권한 부여** → 재커밋 성공.

## 재발방지
- **403은 인가(permission) 신호로 먼저 해석**. 워크플로 순서 탓으로 돌리기 전에 계정 역할부터 점검.
- 신규 앱 첫 배포 전 서비스계정 권한 프리플라이트 체크를 파이프라인에 추가 검토.

## 관련
- 공용 패턴: [[symptom-is-not-root-cause]]
- 과거 비공유 식별자: `play-commit-draft-permission`, `store-launch-blocker-2026-07-22` (역사 기록, 기준 정보 아님)
