---
type: incident
schema_version: 1
project: game-factory
component: environment
status: resolved
root_cause_status: confirmed
게임: 공통
분류: 환경설정
심각도: 높음
발견일: 2026-07
해결일: 2026-07
상태: 해결
aliases: [에뮬레이터 한글경로 부팅패닉, vbmeta 한글경로]
태그: [android, emulator, 한글경로, windows]
---

# Android 에뮬레이터가 한글 SDK 경로에서 부팅 패닉 (vbmeta 파라미터 로드 실패)

## 증상
API 36.1 AVD가 부팅 시 패닉. 에뮬레이터가 **vbmeta 파라미터를 읽지 못함**.

## 근본원인
SDK 경로에 **한글이 포함**되어 에뮬레이터가 경로 인코딩을 처리하지 못함
(Windows + 한글 사용자명/경로의 구조적 리스크).

## 수정
`C:\Users\Public\android-sdk` **정션(junction)**을 만들어 ASCII 경로로 에뮬레이터 실행.

## 재발방지
- Android / Node 계열 툴은 **한글 경로 인코딩 오류를 1순위로 의심**(CLAUDE.md 환경 규칙).
- SDK / 빌드 경로는 처음부터 ASCII 정션으로 고정.

## 관련
- 공용 패턴: [[windows-ascii-tool-paths]]
- 과거 비공유 식별자: `android-emulator-korean-path` (역사 기록, 기준 정보 아님)
