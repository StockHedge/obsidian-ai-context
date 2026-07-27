---
type: incident
schema_version: 1
project: game-factory
component: shared-ads
status: resolved
root_cause_status: confirmed
게임: 공통
분류: 코드버그
심각도: 높음
발견일: 2026-07-21
해결일: 2026-07-21
상태: 해결
aliases: [보상형 광고 컨티뉴 미작동, reward.type 빈 문자열 버그]
태그: [admob, rewarded, capacitor, 베타버그]
---

# 보상형 광고 reward.type='' → 컨티뉴(부활) 미작동

**게임**: 공통(광고 서비스, order-pop에서 실증) · **파일**: `src/services/ads.native.js`

## 증상
베타 버그 #1/#2. 보상형 광고를 **끝까지 시청하고 "Reward granted" 로그까지 떴는데도**
게임이 이어지지 않고 게임오버 화면이 그대로 유지. `continueGame()`이 아예 호출되지 않음.

## 근본원인
`@capacitor-community/admob`의 `showRewardVideoAd()`가 resolve하는
`AdMobRewardItem`은 `{ type: string, amount: number }`. Google **테스트** 보상형 광고는
`reward.type`을 **빈 문자열('')**로 주는 경우가 있다. 기존 판정이
`Boolean(result?.type)`만 봤기 때문에 `Boolean('') === false` →
완주·보상 지급됐음에도 `rewarded=false` → main.js가 컨티뉴를 건너뜀.

## 수정 (전/후)
```js
// before
const rewarded = !!result && Boolean(result?.type);
// after — type이든 amount든 하나라도 채워지면 보상 인정
const rewarded = !!result && (Number(result.amount) > 0 || Boolean(result.type));
```
중도 이탈 시엔 `amount<=0 && type=''` → false, 또는 플러그인 reject → main.js catch로 처리.

## 재발방지
- 서드파티 SDK의 "성공" 판정은 **truthy 1개 필드**가 아니라 **의미 필드 복수**로 판단.
- 테스트 광고와 실 광고의 반환 형태 차이를 항상 의심(빈 문자열 / 0 케이스).

## 관련
- `src/services/ads.native.js` `showRewarded()` — 해당 주석에 근본원인 상세 기록됨.
