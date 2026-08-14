---
type: incident
schema_version: 1
project: ai-video-factory
component: git
status: resolved
root_cause_status: unknown
분류: 데이터 손실·버전관리
심각도: 높음
발견일: 2026-08-12
해결일: 2026-08-12
상태: 재발방지 적용 (손실분 복구 불가)
aliases: [git 이력 리셋, 원격 백업 부재, codex 재개편 이력 소실]
태그: [git, 백업, 데이터손실, ai-인수인계]
agents: [claude-code]
---

# v1 개발 이력이 git에서 사라진 채 원격 백업도 없었다

**대상**: `ai-video-factory` 저장소. 2026-03~07 약 4개월간의 v1 개발 이력.

## 증상

2026-08-12 세션 시작 시 저장소를 실측하니 커밋이 4개뿐이었다.

```
6950386  2026-08-11 17:37  feat: rebuild local video factory v2  ← initial commit (145 files)
061ebd3  2026-08-11 21:08  fix: synchronize captions and render real physics
71ecc44  2026-08-12 00:37  docs: record qualitative v2 review
b26e927  2026-08-12 02:08  feat: implement v2.2 source-first news slice
```

`main` 브랜치 없음. `git reflog`에도 `HEAD@{2026-08-11 17:37} commit (initial)` 이전 기록 없음.
`git remote -v` 비어 있음. 즉 **4개월치 v1 개발 이력이 어디에도 남아 있지 않다.**

v1 **코드 파일 자체는 온전하다**(`pipeline/*.py` 7,874 LOC). 소실된 것은 커밋 이력이다.

## 근본원인

**미확인.** 두 가설을 구분할 증거가 없다.

1. codex 재개편(2026-08-11 17:37 시작) 시 기존 `.git`을 제거하고 새로 `init`했다
2. 애초에 이 프로젝트는 git 저장소가 아니었고 codex가 처음 도입했다

reflog는 새 `.git`과 함께 생성되므로 (1)이든 (2)든 동일하게 보인다.
프로젝트 문서·메모리에도 2026-08-11 이전의 git 사용 흔적이 없어 (2)의 개연성도 낮지 않다.
**추정을 확정으로 기록하지 않는다.**

## 실제 손실

- 4개월간의 커밋 단위 변경 이력, 커밋 메시지에 담긴 의사결정 맥락
- `git blame`·`git bisect` 능력 (버그의 도입 시점 추적 불가)
- 중간 시점으로의 롤백 능력

## 대응

- 2026-08-12 `gh repo create StockHedge/ai-video-factory --private --source=. --push` 실행.
  push 전 `gitleaks detect` 전체 스캔으로 누출 0건 확인
- 손실된 이력은 복구하지 않았다 — 복구할 원본이 존재하지 않는다

## 재발방지

- **AI 도구를 갈아타기 전에 원격 백업이 있는지 먼저 확인한다.** 다른 도구의 "재개편"은
  파일뿐 아니라 버전관리 자체를 갈아엎을 수 있고, 로컬 `.git`만으로는 이를 막지 못한다
- 세션 시작 훅이 "원격: 없음 (이 머신에만 존재 - 백업 부재)"를 이미 출력하고 있었다.
  **경고는 있었으나 행동으로 이어지지 않았다** — 경고 문구보다 실제 조치가 중요하다
- 원격이 없는 저장소를 발견하면 그 세션 안에서 생성한다. 나중으로 미루지 않는다

## 관련

- 프로젝트 카드: [[PROJECT-CARD]]
- 이후 결정: 저장소 `docs/ai/decisions/ADR-0001-hybrid-v2-comfyui-autonomy.md`
