---
type: incident
schema_version: 1
project: game-factory
component: order-pop
status: resolved
root_cause_status: suspected
게임: order-pop
분류: 배포·콘솔
심각도: 보통
발견일: 2026-07-22
해결일: 2026-07-22
상태: 해결
aliases: [등재정보 사전검사 오탐, 빠른검사 기능묘사 차단]
태그: [play-console, 등재정보, 사전검사, 오탐]
---

# 게시 개요 "빠른 검사"가 등재정보 기능묘사를 하드블록 (오탐 성격)

**게임**: order-pop (block-clear는 동일 파이프라인에서 통과)

## 증상
production 제출 직전 "빠른 검사"가 **등재정보의 기능 묘사가 불명확**하다며 하드블록.
빨간 배너 "문제 1개가 모든 변경사항에 영향".

## 근본원인
사전검사 휴리스틱의 등재정보 품질 판정. block-clear는 통과한 것으로 보아 **오탐 가능성**.
실제 정책 위반이라기보다 문구 표현에 대한 자동 판정으로 추정.

## 수정
제목 · 간단한 설명을 **기능이 분명히 드러나게** 수정 후 재시도 → 통과.
(등재정보 원문: `order-pop/store-assets/listing.*.md`)

## 재발방지
- 사전검사 차단은 **원인이 2종**(이 오탐 vs AD_ID 미선언) — "문제 보기"로 반드시 구분.
- 간단한 설명 / 제목에 핵심 동사 · 목표를 직접 서술(모호한 카피 회피).

## 관련
- 공용 패턴: [[symptom-is-not-root-cause]]
- 과거 비공유 식별자: `play-prereview-listing-quality-block` (역사 기록, 기준 정보 아님)
