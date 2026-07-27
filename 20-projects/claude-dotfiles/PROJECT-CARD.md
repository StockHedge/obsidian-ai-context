---
type: project-card
schema_version: 2
project_id: claude-dotfiles
project: Claude Dotfiles
status: active
repo_path: C:\Users\jihon\.claude
remote_url: https://github.com/StockHedge/claude-dotfiles.git
branch: main
head_commit: ded36a3
updated: 2026-07-27
last_verified: 2026-07-27
agents: [claude-code]
tags: [dotfiles, claude-code, config, windows]
---

# Claude Dotfiles — 프로젝트 카드

## 한 줄 목적

`~/.claude`의 이식 가능한 자산만 버전 관리해 여러 기기에서 동일한 Claude Code 전역 설정을 재현한다.

## 기준 위치

- 로컬 저장소: `C:\Users\jihon\.claude`
- 원격 저장소: `https://github.com/StockHedge/claude-dotfiles.git` (private)
- 현재 브랜치: `main`
- 확인한 HEAD: `ded36a3`
- 프로젝트 `AGENTS.md`: 2026-07-27 v2 마이그레이션으로 신설
- 현재 상태 `docs/ai/NOW.md`: 2026-07-27 신설 (실측 Git·검증 기준)
- 사용자 운영 문서: `README.md`

프로젝트 저장소의 코드·Git·최신 검증 결과가 이 카드보다 우선한다.

## 구조 지도

- `CLAUDE.md`: 전역 지침 겸 이 저장소의 프로젝트 지침 (동일 파일 — ADR-0001)
- `AGENTS.md`: 모든 AI가 공유하는 실행·검증·Git 규칙
- `settings.json`: 권한, env, 훅, statusline, 플러그인
- `agents/`, `skills/`, `hooks/`, `templates/`: 사용자 정의 자산
- `_dotfiles/`: 동기화·부트스트랩·pre-commit 가드
- `docs/ai/`: AI 공용 컨텍스트

## 핵심 제약

- 이 디렉터리는 Claude Code 전역 설정 디렉터리이면서 저장소 루트다. 전역 지침과
  프로젝트 지침이 같은 파일이므로 `@AGENTS.md` 임포트가 모든 프로젝트 세션에 로드된다.
- `.gitignore`는 deny-all + 화이트리스트다. 새 자산은 명시 등록해야 추적된다.
- 훅 경로는 `$env:USERPROFILE` 기반을 유지한다. 절대경로화하면 타 기기에서 조용히 깨진다.
- 자격증명은 `.gitignore` denylist와 `_dotfiles/pre-commit`으로 2중 차단한다.
- 빌드·테스트 러너가 없다. 검증은 JSON 파싱·PowerShell 구문 검사·훅 실동작 관찰로 한다.

## 현재 단계

2026-07-27 공유 AI 컨텍스트 프로토콜 v2를 적용했다. 관리 블록 병합, `AGENTS.md`와
`docs/ai/` 신설, `.gitignore` 화이트리스트 확장까지 완료했고 **커밋하지 않았다**.
저장소에 작성자가 다른 미커밋 변경 12건이 함께 있어 임의로 섞지 않았다.

## 중요 사건

- 현재 Vault에 이 프로젝트 전용 사건 노트 없음
- 저장소 내 이력: 2026-07-16 훅 경로 절대경로화 회귀 (`README.md`에 기록, 복구됨)

## 주요 마일스톤

- 2026-07-27 공유 AI 컨텍스트 프로토콜 v2 적용 (미커밋, 저장소 `docs/ai/NOW.md` 참조)

## 관련 공통 패턴

- 현재 연결된 공통 패턴 없음

## 마지막 검증

- 확인일: 2026-07-27
- 확인한 근거: Git 상태·브랜치·HEAD, `.gitignore` check-ignore 결과,
  `settings.json` JSON 파싱, 추적 대상 `.ps1` 6종 구문 검사, 훅 경로 실존 확인
- 확인한 작업 트리: 이번 세션 외 미커밋 변경 12건 존재, 건드리지 않음
- 확인하지 못한 항목: 새 Claude Code 세션에서 `@AGENTS.md` 임포트가 실제로
  인라인되는지 (현재 세션이 편집 이전에 시작되어 확인 불가)
- 미결정 항목: `scripts/` 추적 누락 (저장소 `docs/ai/BACKLOG.md` 1번)
