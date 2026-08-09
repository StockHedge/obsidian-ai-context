---
type: runbook
schema_version: 2
policy_version: 1
updated: 2026-08-09
---

# 노트북 초기 설정 절차

메인 PC에서는 2026-08-09에 설정이 끝났다. 이 문서는 **노트북(`DESKTOP-MEIMN4T`)에서 한 번만**
실행하는 절차다. 운용 규칙은 [[VAULT-SYNC]]에 있다.

노트북에서 Claude Code를 쓴다면 이 문서를 그대로 읽혀 실행시켜도 된다.

## 0. 기존 노트북 Vault 처리 (중요)

노트북에는 이미 `Obsidian Vault`가 있고 그 안에 `AI-CONTEXT-LOGGING` 사본이 들어 있다.
그 사본의 작업분(2026-07-31 ~ 08-04, 8커밋)은 2026-08-09에 원격으로 전부 병합됐다
([[audits/2026-08-09-vault-consolidation]]). `AIWebbuilder/`와 카페24 레퍼런스도 통합됐다.

**따라서 기존 노트북 Vault는 더 이상 기준이 아니다.** 아래 순서를 지킨다.

1. 기존 `Obsidian Vault` 폴더를 통째로 ZIP 백업한다.
2. Obsidian에서 해당 Vault 등록을 해제한다(폴더는 아직 지우지 않는다).
3. 3절대로 새로 clone한 폴더만 연다.
4. 2~3주 운용해 문제가 없으면 백업만 남기고 원본 폴더를 정리한다.

두 폴더를 동시에 열어두면 같은 노트가 두 벌 존재하게 되고, 이번과 같은 분기가 반복된다.

## 0-1. 전제 확인

```powershell
git --version                                  # 없으면 https://git-scm.com/download/win
Test-Path 'C:\Program Files\Obsidian\Obsidian.exe'   # 없으면 Obsidian 설치
gh --version                                   # 없어도 되지만 있으면 인증이 편하다
```

## 1. Git 전역 설정

한글 파일명과 깊은 경로 때문에 기본값으로 두면 `git status` 출력이 깨지고 체크아웃이
실패할 수 있다. PC와 동일하게 맞춘다.

```powershell
git config --global user.name  "jihon"
git config --global user.email "jihono55@gmail.com"
git config --global core.quotepath false
git config --global core.longpaths true
git config --global core.autocrlf false
git config --global i18n.logOutputEncoding utf-8
```

`core.longpaths`는 Windows 레지스트리도 함께 켜져 있어야 완전히 동작한다.

```powershell
# 관리자 PowerShell
Set-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem' -Name LongPathsEnabled -Value 1
```

## 2. GitHub 인증

```powershell
gh auth login          # 브라우저 로그인. 계정: StockHedge / 프로토콜: HTTPS
```

`gh`가 없으면 3번의 `git clone` 실행 시 Git Credential Manager 창이 뜬다. 거기서 브라우저
로그인하면 Windows 자격 증명 관리자에 저장된다. **GitHub 비밀번호가 아니라 브라우저 로그인
또는 개인 액세스 토큰**이 필요하다.

## 3. Clone

경로는 PC와 동일하게 맞추는 것을 권장한다. `50-agent-config`의 배포 절차가 Vault 절대경로를
참조할 수 있고, 두 기기의 경로가 같으면 문서에 경로를 조건부로 쓸 필요가 없다.

```powershell
$parent = "$env:USERPROFILE\TheNewProject\OBSIDIAN"
New-Item -ItemType Directory -Force -Path $parent | Out-Null
Set-Location $parent
git clone https://github.com/StockHedge/obsidian-ai-context.git AI-CONTEXT-LOGGING
Set-Location AI-CONTEXT-LOGGING
git config pull.rebase false
git log --oneline -3
```

2026-08-09 실측: 노트북의 사용자 프로필은 `jihon`이 아니라 **`강지호`** 다.
따라서 실제 경로는 `C:\Users\강지호\TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING` 이다.
Vault 폴더 구조는 두 기기가 같지만 프로필 이름이 다르므로, 문서·스크립트에는 절대경로를
하드코딩하지 말고 `%USERPROFILE%` 기준으로 쓴다.

## 4. Obsidian에서 열기

Obsidian → `Open folder as vault` → 위에서 clone한 `AI-CONTEXT-LOGGING` 폴더 선택.

새 폴더를 만들어 vault를 생성하지 말 것. clone한 폴더를 그대로 여는 것이 맞다.

## 5. Git 플러그인 설치

`설정 → Community plugins → Restricted mode 해제 → Browse → "Git" 검색 → 제작자 Vinzent03
→ Install → Enable`

## 6. 플러그인 설정

[[VAULT-SYNC]]의 「플러그인 설정값」 표를 그대로 적용한다. 도입 1주차 열을 먼저 쓴다.

노트북에서만 다른 값 하나:

- `{{hostname}} placeholder replacement` = `laptop`  (PC는 `pc-main`)

이 값은 저장소에 커밋되지 않고 기기 로컬에 저장되므로 기기마다 직접 지정해야 한다.

명령 팔레트에서 `Obsidian Git: Pull`과 `Obsidian Git: Commit-and-sync`에 단축키를 지정한다.

## 7. 왕복 검증

한 번은 반드시 양방향을 확인하고 끝낸다.

1. 노트북에서 `10-inbox/README.md` 맨 아래에 한 줄 추가 → `Commit-and-sync`
2. PC에서 Obsidian 열기 → `Pull` → 그 줄이 보이는지 확인
3. PC에서 그 줄 삭제 → `Commit-and-sync`
4. 노트북에서 `Pull` → 삭제가 반영됐는지 확인

여기까지 통과하면 설정이 끝난 것이다. 실패하면 상태 표시줄의 오류 메시지를 그대로 기록해
`10-inbox/`에 남긴다.

## 8. 하지 말 것

- 노트북 Vault 저장소에 `.git/autosync` 파일을 만들지 않는다.
  이 저장소의 자동 커밋 주체는 Obsidian Git 단독이다 ([[GIT-POLICY]] 「Vault 적용」).
- PC와 노트북에서 동시에 Vault를 열어두고 양쪽 다 편집하지 않는다.
- 충돌 시 `Discard all changes`를 쓰지 않는다.
