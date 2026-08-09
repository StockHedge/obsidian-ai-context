---
type: operating-policy
schema_version: 2
policy_version: 1
updated: 2026-08-09
---

# Vault 기기 간 동기화 운용 규칙

메인 PC(`pc-main`)와 노트북(`laptop`) 사이에서 이 Vault를 어떻게 주고받는지를 정한다.
Git 권한 경계는 [[GIT-POLICY]], 커밋 단위와 기준선은 [[VERSION-CONTROL]], AI 간 컨텍스트
공유 한계는 [[SYNC-BOUNDARIES]]에 있다. 이 문서는 **사람의 일상 조작 절차**만 다룬다.

## 구성

- 원격: GitHub Private 저장소 `StockHedge/obsidian-ai-context`
- 경로: 두 기기 모두 `%USERPROFILE%\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
  - PC(`pc-main`): `C:\Users\jihon\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
  - 노트북(`laptop`): `C:\Users\강지호\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING`
  - **사용자 프로필 이름이 두 기기에서 다르다**(`jihon` / `강지호`). 절대경로를 문서나
    스크립트에 하드코딩하지 말고 `%USERPROFILE%` 기준으로 쓴다. 노트북 경로에는 한글이
    들어가므로 `core.quotepath false`·`core.longpaths true` 설정이 특히 중요하다
    ([[windows-ascii-tool-paths]] 참조).
- 도구: Obsidian 커뮤니티 플러그인 `Git` (Vinzent03)
- 인증: Git Credential Manager (Windows 자격 증명 관리자)
- 전송 경로는 GitHub이며 Tailscale과 무관하다. Tailscale/Remote-SSH는 코드 작업용이고
  GUI 앱인 Obsidian은 그 경로로 열 수 없다.

## 원칙

**열 때 Pull, 떠날 때 Commit-and-sync.**

## 1단계 — 도입 1주차 (자동화 없이)

1. Obsidian을 연다.
2. 명령 팔레트에서 `Obsidian Git: Pull`을 직접 실행한다.
3. 작업한다.
4. 자리를 뜨기 전 `Obsidian Git: Commit-and-sync`를 실행하고 Obsidian을 닫는다.

두 명령에 단축키를 지정한다. 이 기간의 목적은 동기화가 **언제 실패하는지**를 눈으로
확인하는 것이다. 처음부터 자동화를 켜면 실패가 조용히 쌓인다.

## 2단계 — 안정화 이후 (자동화 켠 뒤)

- 열 때: 아무것도 하지 않는다. `Pull on startup`이 처리한다.
  상태 표시줄에 오류가 뜨면 **그 자리에서** 해결하고 편집을 시작한다.
- 작업 중: 15분 주기와 편집 정지 시점에 자동으로 커밋·푸시된다.
- 떠날 때: 그래도 `Commit-and-sync`를 한 번 누른다.
  마지막 자동 sync 이후의 구간을 덮는 방법은 이것뿐이며 자동화로 대체되지 않는다.

즉 외울 규칙은 **"떠날 때 단축키 한 번"** 하나로 줄어든다.

## 플러그인 설정값 (두 기기 동일)

| 설정 | 1주차 | 안정화 이후 |
|---|---|---|
| `Pull on startup` | ON | ON |
| `Auto commit-and-sync interval (minutes)` | 0 | 15 |
| `Auto commit-and-sync after stopping file edits` | OFF | ON |
| `Auto pull interval (minutes)` | 0 | 0 |
| `Push on commit-and-sync` | ON | ON |
| `Merge strategy` | Merge | Merge |

- `Commit message on auto commit-and-sync`: `vault: {{date}} {{hostname}} ({{numFiles}} files)`
- `{{date}} placeholder format`: `YYYY-MM-DD HH:mm:ss`
- `{{hostname}} placeholder replacement`: PC는 `pc-main`, 노트북은 `laptop`
  → 이력만 보고 어느 기기의 변경인지 즉시 구분된다. 충돌 추적에 결정적이다.
- `Auto pull interval`을 끄는 이유: 편집 중 원격 변경이 비동기로 끌려오면 사용자가 인지하지
  못한 상태로 충돌에 진입한다. 시작 시 pull + 수동 pull이면 1인 2기기에서는 충분하다.
- `Merge strategy`를 `Rebase`로 두지 않는 이유: 충돌 시 rebase 중단 상태에 갇히기 쉽고,
  Vault에는 선형 이력의 가치가 낮다.

## 상황별

- **노트북을 들고 나갈 때**: PC에서 Commit-and-sync 후 **PC의 Obsidian을 닫는다.**
  켜둔 채 나가면 PC에서 자동 커밋이 계속 돌아 노트북에서 다시 Pull해야 한다.
- **오프라인**: 그대로 쓴다. 커밋은 로컬에 쌓이고 push만 실패한다.
  온라인이 되면 Commit-and-sync 한 번으로 밀려 올라간다.
- **RDP로 PC 화면에 붙어 편집할 때**: PC 쪽 사본을 만지는 것이므로 노트북 로컬 사본과
  갈라진다. 세션을 끊기 전 Commit-and-sync, 노트북에서 Pull.
  ※ 2단계 자동화를 켜면 이 규칙은 마지막 sync 이후 구간에만 유효한 한시 규칙이 된다.
- **모바일**: 이 구조에 포함되지 않는다. 플러그인 개발자가 모바일 Git 구현을
  "very unstable"이라며 직접 비권장하고, Android/iOS에서는 네이티브 git과 SSH 인증을
  쓸 수 없다. 모바일이 필요해지면 Obsidian Sync로 전환한다.

## 금지

**양쪽 기기에서 동시에 열어두고 둘 다 편집하지 않는다.**
충돌이 발생하는 시나리오는 사실상 이것 하나다.
PowerToys Mouse Without Borders로 두 화면을 함께 쓰고 있어 어기기 쉽다.

## 충돌 처리

1. 상태 표시줄에 오류가 뜨고 파일 안에 `<<<<<<<` 마커가 생긴다.
2. 필요한 내용을 하나로 합친다.
3. `<<<<<<<`, `=======`, `>>>>>>>` 표식을 모두 지운다.
4. 저장한 뒤 `Commit-and-sync`를 다시 실행한다.

- `Discard all changes`는 **쓰지 않는다.** 로컬 수정본이 복구 불가로 사라진다.
- `PROGRESS.md`, `SYNC-STATUS.md`, `NOW.md` 류 인계 문서가 충돌하면 어느 한쪽을 버리지 않고
  양쪽을 병합한다 ([[GIT-POLICY]] 「충돌 시」와 동일).

## 자동화 소유권

이 저장소의 자동 커밋·푸시 주체는 **Obsidian Git 단독**이다.

- Vault 저장소에는 `.git/autosync` 마커를 **만들지 않는다.**
  Claude Code의 `Stop` 훅 자동 커밋·푸시 대상에서 제외한다.
- 근거: 같은 워킹트리에 자동 커밋 주체가 둘이면 `index.lock` 경합과 편집 중 부분 커밋이
  발생하고, 누가 만든 커밋인지 구분되지 않는다.
- 반대 방향(Claude Code 훅에 소유권을 주고 플러그인 자동화를 끄는 안)은 채택하지 않는다.
  노트북에서 Obsidian만 켜고 Claude Code를 쓰지 않는 경우 자동 백업이 통째로 사라진다.
- Claude Code가 Vault를 편집할 때는 [[GIT-POLICY]]의 수동 커밋 규칙을 따른다.

## 비밀값

- `.gitignore`가 `.obsidian/plugins/*/data.json`을 **기본 차단**한다.
  일부 플러그인은 이 파일에 API 키와 개인키를 평문으로 저장한다.
- 공유하려면 해당 플러그인만 `!` 규칙으로 개별 해제하되, **해제 전 파일 내용을 직접 확인**한다.
- GitHub의 secret scanning·push protection은 public 저장소에서만 무료이고 개인 소유 private
  저장소에는 적용되지 않는다. 방어선은 전적으로 로컬(`.gitignore` + 커밋 전 확인)이다.
