---
게임: (전체 5개 앱)
분류: 출시
심각도: Critical
발견일: 2026-07-29
해결일: 2026-07-29
상태: 수정완료
aliases: [Play 전면 거부, 기본 런처 아이콘, 스토어 등록정보 불일치, 혼동을 야기하는 주장]
---

# Play 전면 거부 — 5개 앱 전부 Capacitor 기본 런처 아이콘인 채 제출됐다

## 증상

2026-07-29, 심사 중이던 5개 앱(블록 클리어·순서 팝·탱글 아웃·알케미 바운스 마스터·
협동 타워 디펜스)이 **전건 동일 사유로 거부**됐다: "혼동을 야기하는 주장 정책 위반 —
앱 스토어 등록정보 불일치. 앱의 설치된 아이콘 또는 이름이 스토어 등록정보와 다릅니다."
증거로 `LAUNCHER_ICON`/`IN_APP_EXPERIENCE` 스크린샷과 `HI_RES_ICON`(ko-KR) 쌍이 첨부됐다.

## 근본원인

**파이프라인 어디에도 런처 아이콘을 교체하는 단계가 없었다.**

- `cap add android`는 Capacitor 기본 로봇 아이콘을 깐다.
- `gen-store-graphics.mjs`는 **스토어 등재용** icon-512만 생성한다 — APK 안의
  mipmap은 건드리지 않는다.
- 두 산출물이 완전히 분리돼 있어 "스토어에는 예쁜 아이콘, 설치하면 로봇"이
  구조적으로 보장됐다. 실측: 5개 게임의 `mipmap-xxxhdpi/ic_launcher.png` md5가
  전부 동일(`9e029293ab...` = 기본 아이콘).

설치명은 원인이 아니었다 — 5개 모두 ko 로케일 `app_name`이 스토어 제목과
일치(전체 또는 접두)함을 확인했다. 아이콘 단독 위반이다.

## 수정 (2026-07-29)

- `game-factory/tools/gen_launcher_icons.py` 신설 — store-assets의 icon-512를
  원본으로 legacy/round/adaptive-foreground 15개 PNG + 배경색(아이콘 테두리
  중앙값)을 굽는다. 5개 게임 전부 적용.
- versionName 상향(1.0→1.0.1, coop 1.2→1.2.1) + listing 4언어 출시 노트 추가
  → `play-release.mjs`로 5개 재빌드·프로덕션 재업로드 → 게시 개요에서 재제출.
- 검증: AAB 내부 아이콘이 소스와 **픽셀 단위 동일**(ImageChops bbox=None —
  md5 비교는 AAPT2 재압축 때문에 항상 다르게 나온다, 함정), 실 광고 ID 포함,
  jarsigner verified.

## 재발방지

- **`play-release.mjs`에 기본 아이콘 차단 가드** — 업로드 전에
  `mipmap-xxxhdpi/ic_launcher.png` md5가 기본 아이콘과 같으면 실패시킨다.
  아이콘 교체를 잊어도 이제 업로드 자체가 안 된다.
- 거부됨/초안 상태 앱의 `edits.commit`은 `changesNotSentForReview=true`가
  필요하다(HTTP 400로 거절됨) — play-release가 자동 폴백하도록 수정. 커밋 후
  **게시 개요 "변경사항 전송"은 여전히 사람/브라우저 몫**이다.
- 교훈: 스토어 자산과 APK 자산처럼 **같은 정보가 두 산출물로 갈라지는 지점**은
  전부 정합 검사 대상이다. 설치명(strings.xml ↔ 스토어 제목)은 이번엔 정합했지만
  같은 구조의 위험이 있다.

## 관련

- [[2026-07-28-play-survey-coordinate-misclick]] — 같은 제출 파이프라인의 앞 단계 사고
- [[2026-07-27-registry-forgery-stopgap]] — "산출물 이중화 → 불일치" 계열의 다른 사례
