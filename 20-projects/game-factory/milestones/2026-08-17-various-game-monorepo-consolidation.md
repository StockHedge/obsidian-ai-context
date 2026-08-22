---
type: milestone
schema_version: 1
project: game-factory
component: various-game
status: monitoring
verified: 2026-08-21
agents: [claude-code]
source_repo: StockHedge/various-game
source_commit: d74f764
tags: [monorepo, git, branch-convention, crlf, subtree-hash]
---

# various-game — 게임 저장소 10개를 모노레포 1개로 통합

## 무엇을 달성했나

2026-08-17, 게임별 독립 저장소 10개를 `StockHedge/various-game`(private) 하나로
통합했다. 커밋 420건·파일 1428개가 **원본 해시를 보존한 채** 이관됐고, 구 저장소
10개는 archive 처리했다. 통합 시점에 게임 8종이 인식된다.

2026-08-21, 공장 파이프라인이 산출물을 모노레포에 반영하는 절차를 코드로 고정했다
(`factory/vcs.py`, 커밋 `d74f764`).

## 브랜치 규약 (2026-08-21 `game/cross` 추가)

게임별 저장소가 아니므로 **브랜치 이름이 대상을 명시**해야 한다.

```
game/<slug>/<주제>     예) game/matgo/beta-fix
game/cross/<주제>      게임 2개 이상을 함께 고치는 교차 변경
factory/<주제>         game-factory 변경
standards/<주제>       game-standards 변경
```

`cross`는 슬러그가 아니라 **예약어**다. 한 게임만 고치면 반드시 그 게임의 슬러그를
쓴다 — `cross`를 기본값처럼 쓰면 "이름이 대상을 명시한다"는 규칙 자체가 죽는다.

기준 문서는 저장소 `README.md`의 "브랜치 규칙" 절이고, 이 노트는 그 사본이 아니라
왜 그렇게 정했는지를 남긴다. `main`은 보호 브랜치 — push 전 사용자 확인
([[GIT-POLICY]]).

## 모노레포가 바꾼 것 (개별 저장소 시절 전제가 깨지는 지점)

**1. `git add -A` / `commit -am` 금지.** 모노레포에는 다른 세션·사용자 소유의
미커밋 변경이 상시 존재한다. 2026-08-17에 `commit -am`으로 타 세션 소유 14건
(`*/game.json`, `*/android/app/build.gradle`, `web/*`)을 실제로 섞어 커밋했다가
revert했다. 커밋은 **경로 한정 스테이징**으로만 한다.

**2. 잡 성패 판정에 저장소 HEAD를 쓰면 안 된다.** 옆 게임의 커밋만으로 HEAD가
바뀌므로, 아무 일도 안 한 잡이 성공으로 오판된다. 게임별 **서브트리 해시**
(`git rev-parse HEAD:<게임경로>`)를 쓴다.

**3. 게임 폴더 안 `git init` 금지.** 중첩 저장소가 되어 게임 파일이 모노레포
추적 밖으로 밀려난다. 스캐폴드는 모노레포 루트에서 경로 지정 커밋한다.

**4. CRLF stat 캐시 함정.** 통합 직후 494개 파일이 modified로 보였다. 원인은
전환이 아니라 **개별 저장소 시절부터 있던 워크트리 CRLF / 블롭 LF 불일치**가
각 저장소의 `.git/index` stat 캐시에 가려져 있던 것. 블롭 SHA 동일·`diff --numstat`
비어 있음으로 확인했고, `core.autocrlf=true` + `git add -u` 리프레시 + 루트
`.gitattributes`로 정리했다. 실제 변경은 14건뿐이었다.

## 검증

- `factory/vcs.py` 회귀 테스트 7건 포함 공장 테스트 73건 통과 (2026-08-21)
- 실제 모노레포 대조: `repo_root` == `GAME_ROOT`, 게임별 트리 해시 상이,
  중첩 저장소 0건

## 비용

GitHub private 저장소는 저장소 수·용량과 무관하게 무료 — 통합에 별도 비용 없음.

## 관련

- 저장소: `README.md`, `.gitattributes`, `factory/vcs.py`, `tests/test_vcs.py`
- 커밋: `d74f764`
- [[2026-08-21-verification-bypass-blind-spot]]
