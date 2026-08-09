---
type: cross-project-pattern
schema_version: 1
status: verified
created: 2026-08-04
updated: 2026-08-04
source_projects: [for-marketing]
agents: [claude-code]
tags: [object-storage, s3-api, cloudflare-r2, cloud-integration]
---

# S3 호환 스토리지는 "호출 성공"과 "의미 동일"이 다르다

## 문제

S3 호환을 표방하는 오브젝트 스토리지(Cloudflare R2 등)에 S3 관례를 그대로 적용하면,
API 호출 자체는 에러 없이 끝나거나 인자가 조용히 무시되는데 실제로는 의도한 동작이
일어나지 않을 수 있다. for-marketing에서는 업로드를 `ACL=public-read`로, 회수를
`put_object_acl(ACL=private)`로 구현했다. 업로드는 통과했지만(R2가 ACL 인자를 조용히
무시) 회수가 `NotImplemented`로 실패했다 — R2는 객체별 ACL 개념 자체가 없고 공개
여부는 버킷 단위(r2.dev/커스텀 도메인)로만 결정되기 때문이다. 결과적으로 "지연
비공개" 기능이 통째로 무동작이었는데도 업로드 경로만 봐서는 드러나지 않았다.

## 재사용 가능한 원칙

- "S3 호환"은 자주 쓰는 하위 오퍼레이션 집합에 대한 호환이지, 전체 API 표면의 의미적
  동일성을 보장하지 않는다.
- 접근 제어가 **객체 단위**인지 **버킷 단위**인지를 코드 작성 전에 그 서비스 공식
  문서에서 먼저 확인한다.
- 무시되는 인자(silently-ignored parameter)와 미구현 오퍼레이션(hard failure)이 같은
  SDK 안에 공존할 수 있다 — "예외 없이 끝났다"를 "의도대로 됐다"의 증거로 쓰지 않는다.
- 지연 비공개·회수가 필요한 설계에서 대상 서비스가 객체 ACL을 지원하지 않는다면,
  ACL 변경이 아니라 **객체 삭제**(또는 별도 비공개 경로/버킷으로 이동)로 구현한다.

## 적용 조건

- Cloudflare R2, Backblaze B2 등 "S3 호환 API"를 표방하는 오브젝트 스토리지를 boto3
  등 S3 클라이언트 SDK로 다루는 모든 코드.
- 업로드 시점과 접근 제어 변경 시점이 분리된 설계(예: "N일 후 비공개 전환", "임시
  공개 URL").
- for-marketing·embolos·aiwebbuilder 모두 R2를 사용하므로 세 프로젝트 전부가 적용
  대상.

## 적용하지 말아야 할 조건

- 실제 AWS S3를 쓰는 경우는 해당 없음(객체 ACL이 정상 동작한다).
- 버킷을 항상 완전 공개 또는 항상 완전 비공개로만 쓰고 객체별 전환이 없는 설계라면
  이 함정 자체가 발생하지 않는다.

## 확인 절차

1. 스토리지 서비스의 공식 문서에서 "S3 대비 미지원/상이 오퍼레이션" 목록을 먼저
   확인한다.
2. 실버킷에 업로드 → 공개 URL GET 200 확인 → 비공개 전환(또는 회수) 오퍼레이션
   실행 → 같은 URL GET이 403/404로 바뀌는지까지 실왕복으로 검증한다. 코드 정적
   검토나 SDK 호출의 "성공" 리턴값만으로 판정하지 않는다.
3. 왕복이 실패하면 "인자가 무시됐는지" vs "오퍼레이션 자체가 미구현인지"를 응답
   (또는 무응답·예외 타입)으로 구분해 원인을 좁힌다.

## 근거 사건

- for-marketing `backend/app/intelligence/media/storage.py` — 회수 로직을 ACL
  변경에서 객체 삭제로 교체(커밋 98476d7, R2 실연결 시점에 발견·수정).
