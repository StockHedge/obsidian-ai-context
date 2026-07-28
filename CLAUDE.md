# CLAUDE.md

## 프로젝트 개요
- 무엇: Obsidian 볼트 — AI 작업 컨텍스트 로깅/축적용 노트 저장소
- 성격: 코드 프로젝트가 아니라 **문서 저장소**. 빌드·실행 명령 없음
- 구조 (Johnny.Decimal 계열 번호 체계):
  - `00-home` 진입점 / `10-inbox` 수집 / `20-projects` 프로젝트별 / `30-patterns` 패턴
  - `40-profile` 프로필 / `50-agent-config` 에이전트 설정 / `90-archive` 보관
  - `_templates` 노트 템플릿 / `.obsidian` Obsidian 설정

## 컨벤션
- 브랜치: `main`
- 새 노트는 `_templates`의 템플릿에서 시작하고, 확정 전까지는 `10-inbox`에 둘 것
- 링크는 wikilink(`[[...]]`)를 사용. 파일 이동 시 링크 깨짐 여부를 확인할 것
- `.obsidian/` 설정 변경은 머신 간 충돌이 잦으므로 커밋 여부를 의식적으로 판단할 것

## 원격 저장소
- **GitHub 원격 없음.** 백업이 없는 상태
- 필요 시: `gh repo create ai-context-logging --private --source=. --push`

## 세션 규칙 (중요)
- 세션 시작 시: `PROGRESS.md`를 먼저 읽고 `git status`로 미커밋 노트가 있는지 확인
- 세션 종료 시: `PROGRESS.md` 갱신 → 커밋 (원격이 생기면 `git push`)
