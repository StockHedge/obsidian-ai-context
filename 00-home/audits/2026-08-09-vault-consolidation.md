---
type: audit
schema_version: 2
updated: 2026-08-09
agents: [claude-code]
tags: [vault, consolidation, sync]
---

# 2026-08-09 Vault 통합·원격 연결 검수

## 무엇을 했나

세 갈래로 흩어져 있던 Obsidian 기록을 이 Vault 하나로 합치고, GitHub Private 저장소를
연결해 두 기기가 같은 Vault를 쓰도록 했다.

통합 전 상태:

1. **PC** `Documents\Obsidian Vault` — 258파일 52.86MB. 첨부 PNG 220장(91.9%),
   md 27개. Git 미초기화.
2. **PC** `TheNewProject\OBSIDIAN\AI-CONTEXT-LOGGING` — 67파일 0.17MB.
   로컬 Git(`main`, 8커밋, 원격 없음).
3. **노트북** `Obsidian Vault` — 위 1·2의 사본 + `AIWebbuilder/` 9건.
   내부 `AI-CONTEXT-LOGGING`은 같은 저장소가 8커밋 더 진행된 상태.

## Git 이력 병합

두 기기가 공통 조상 `dbee796`에서 각각 8커밋씩 앞서 있었다. 파일 복사가 아니라
실제 병합으로 처리해 양쪽 이력을 모두 보존했다.

- 노트북 → `772c7c9` (2026-07-31 ~ 08-04): embolos 보안 리뷰·pytest prod DB 니어미스 사건,
  마일스톤 4건, `for-marketing` 프로젝트 신설, 패턴 4건, claude-code inbox 2건,
  cursor 초안 2건의 `90-archive` 이동
- PC → `fcdabff` (2026-08-09): 원격 연결, `VAULT-SYNC`·`LAPTOP-SETUP` 신설,
  `.gitignore`/`.gitattributes` 강화, Obsidian Git 플러그인 커밋
- 병합 커밋 `180a3c5`. 충돌은 `20-projects/embolos/PROJECT-CARD.md` 1건뿐이었다.

### 충돌 해소 근거

노트북의 2026-08-04 상태가 엄밀히 최신이므로 그쪽을 기준으로 삼되, 최신본에서 누락된
백로그 2건(가격·무료 티어 캡·게이팅, 만료 테넌트 purge 조건)을 이월로 명시해 되살렸다.
2026-07-27 검증 상세는 [[90-archive/2026-07-27-cursor-context-v2-apply]]에 남아 있어
카드에서는 참조로 대체했다.

### 삭제 vs 보관

이 PC 세션은 cursor 초안 2건을 삭제 처리했으나, 노트북 쪽은 `90-archive`로 보관했다.
병합 결과는 **노트북의 보관 처리가 우선**한다. git이 rename으로 인식해 이 PC의
내용 수정분도 보관본에 그대로 반영됐다.

## 파일 이관 매핑

| 원본 | 이관 위치 | 건수 |
|---|---|---|
| `Obsidian Vault/*.png` | `_attachments/embolos/cafe24/` | 220 |
| `Obsidian Vault/Embolos/레퍼런스/카페24/*.md` | `20-projects/embolos/references/cafe24/` | 13 |
| `Obsidian Vault/Embolos/*.pdf` | `20-projects/embolos/references/` | 1 |
| `Obsidian Vault/Embolos/생각하고 있는 기획들…md` | `20-projects/embolos/` | 1 |
| `Obsidian Vault/GameFactory-BugLog/logs/*.md` | `20-projects/game-factory/incidents/` | 11 |
| `Obsidian Vault/GameFactory-BugLog/_dashboard.md` | `20-projects/game-factory/BUGLOG-DASHBOARD.md` | 1 |
| 노트북 `AIWebbuilder/**` | `20-projects/aiwebbuilder/` | 9 |
| `Obsidian Vault/환영합니다!.md` | 제외 (Obsidian 기본 노트) | 1 |
| `.obsidian/plugins/obsidian-local-rest-api/**` | 제외 (아래 참조) | 4 |

노트북의 `Embolos/`와 중첩 `Obsidian Vault/`는 PC 사본의 부분집합이었다.
해시 대조 결과 공유 파일은 전부 동일했고, 노트북 쪽에는 첨부 220장과
`2026-07-29-play-rejection-default-launcher-icon.md`가 없었다. PC 사본을 정본으로 썼다.

## 링크 안전성

이관 전 확인한 사항:

- 첨부 참조 220건은 전부 `![[파일명.png]]` 최단 형식이다. Obsidian은 이 형식을
  폴더가 아니라 **파일명으로 해석**하므로 폴더를 옮겨도 링크가 깨지지 않는다.
- AIWebbuilder 내부 링크도 `[[확정-결정]]` 류의 최단 형식이다. 그래서 파일명은
  하나도 바꾸지 않았다. **파일명을 영문 슬러그로 바꾸면 이 링크들이 전부 깨진다.**
- 통합 대상 257개 파일명과 기존 Vault 81개 파일명 사이에 **중복 0건**을 확인했다.
  중복이 있었다면 최단 형식 링크가 모호해졌을 것이다.

## 남은 작업

- 파일·폴더명 영문 슬러그화 — [[WRITING-POLICY]]는 영문 소문자·숫자·하이픈을 우선하라고
  하지만, 위 링크 구조 때문에 이름 변경은 링크 재작성과 반드시 함께 해야 한다. 별도 작업.
- 구 BugLog 11건의 프론트매터 스키마를 [[INCIDENT.template]]에 맞춰 정규화.
- `obsidian-local-rest-api` 플러그인은 저장소에 넣지 않았다. `main.js`가 4MB이고
  clone 부트스트랩에 필수도 아니다. 필요하면 기기별로 설치하고, `data.json`은
  API 키와 RSA 개인키를 평문 보관하므로 `.gitignore` 기본 차단을 유지한다.
- 구 `Documents\Obsidian Vault`는 Obsidian 등록에서 해제하고 백업 후 정리.

## 검증

- 병합 후 충돌 표식 잔존 0건 (플러그인 번들 `main.js` 내부 문자열 제외)
- 비밀값 스캔(개인키·GitHub 토큰·Slack 토큰·AWS 키·JWT·DB URL 패턴) 0건
- `git ls-files` 기준 추적 파일 수와 원격 `origin/main` 해시 일치 확인
