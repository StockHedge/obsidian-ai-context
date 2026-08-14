---
type: project-card
schema_version: 2
project_id: ai-video-factory
project: AI Video Factory
status: active
repo_path: C:\Users\jihon\projects\ai-video-factory
remote_url: https://github.com/StockHedge/ai-video-factory.git
branch: feat/v2-overhaul
head_commit: b26e927
updated: 2026-08-12
last_verified: 2026-08-12
agents: [claude-code, codex]
tags: [youtube-shorts, comfyui, python, windows, automation]
---

# AI Video Factory — 프로젝트 카드

## 한 줄 목적

한국어 YouTube Shorts를 로컬에서 무인 제작·게시한다. 채널 **세상모든이야기**(브랜드 StockHedge).

## 기준 위치

- 로컬 저장소: `C:\Users\jihon\projects\ai-video-factory`
- 원격 저장소: `https://github.com/StockHedge/ai-video-factory.git` (private, **2026-08-12 신설**)
- 현재 브랜치: `feat/v2-overhaul` / 확인한 HEAD: `b26e927`
- 현재 상태: `docs/ai/NOW.md`
- 구조·불변조건·검증 명령: `docs/ai/PROJECT.md` (2026-08-12 신설)
- 설계 결정: `docs/ai/decisions/ADR-0001-hybrid-v2-comfyui-autonomy.md`

프로젝트 저장소의 코드·Git·최신 검증 결과가 이 카드보다 우선한다.

## 구조 지도

- `pipeline/*.py` — v1 레거시 7,874 LOC. 무인 운영·ComfyUI 생성. **롤백 기준선으로 보존**
- `pipeline/v2/` — 현행 6,548 LOC. 계약(Pydantic)·QC·Whisper 자막 정렬 계층
- `control_panel/` — Flask 운영 UI (`http://127.0.0.1:5050/v2`)
- `distribution/youtube_uploader.py` — 업로드. 예약 게시(`publish_at`)와 채널 검증 포함
- `analytics/` — 성과 수집(`perf.db`). `config/` — `factory.toml`·`models.json`
- 산출물은 `var/`(v2) / `output/`(v1). 둘 다 Git 비추적

## 핵심 제약

- **v1과 v2가 공존한다.** v1은 삭제하지 않는다
- 상태 변경은 `pipeline/v2/store.py:19 ALLOWED_TRANSITIONS`를 통해서만. 우회 금지
- 업로드는 반드시 `verify_channel_identity()`(EXPECTED_CHANNEL) 통과 — 채널 오염 사고 이력
- `.env`는 `override=True` 로드. 셸 잔존 변수 우선 사고가 반복됐다
- GPU는 단일 자원. RTX 2060 **12GB**를 ComfyUI(4GB)·Whisper(2.2GB)·Blender(6GB)가 공유,
  worker GPU lease로 직렬화
- YouTube quota 편당 약 1,600 units / 본채널 110k per day

## 현재 단계

2026-08-12 하이브리드 전환 계획(ADR-0001) 승인. **구현 미착수.**
2026-07-10 이후 데일리 운영 정지 상태이며, v2를 뼈대로 두고 v1의 ComfyUI 생성과
무인 운영을 Protocol 구현체로 이식해 일 15편 무인 운영 복귀를 목표한다.

확정 정책: 일 15편(07~20시 매시 14편 + brainrot 13:30) / 근거 게이트 제거 /
ComfyUI 6 : 스톡 5 혼합 / TTS는 TypeCAST → OpenAI 전환 / 사람 승인 0.

## 중요 사건

- [[2026-08-12-git-history-reset-no-remote]] — v1 개발 이력 소실, 원격 백업 부재 (복구 불가)

## 주요 마일스톤

- 2026-08-11~12 codex가 v2 재구축 (6,548 LOC, 테스트 38, mypy strict, QC 22항목)
- 2026-08-12 GitHub private 원격 신설 + 최초 push (gitleaks 누출 0건)
- 2026-08-12 ADR-0001 하이브리드 전환 결정

## 관련 공통 패턴

- 현재 연결된 공통 패턴 없음

## 마지막 검증

- 확인일: 2026-08-12
- 확인한 근거: `git log`·`reflog`·`branch -a` 실측, `gitleaks detect` 전체 스캔(누출 0),
  `nvidia-smi`(RTX 2060 12GB), OpenAI `/v1/models` HTTP 200, ComfyUI 체크포인트 sha256 계산,
  `var/db/factory_v2.sqlite3` 상태 조회, `netstat`로 5050 리스닝 확인
- 확인한 작업 트리: 미커밋 변경 0건
- 확인하지 못한 항목: v2 파이프라인의 실제 1편 E2E 생성(구현 착수 전이라 미실행),
  OpenAI TTS 한국어 실제 음질(모델 목록만 확인)
- 미결정 항목: 노출 이력이 있는 키(TypeCAST/WaveSpeed/FAL) rotate 여부 —
  사용자가 2026-07-06 스킵 선택했고 이후 재검토 없음
