---
id: endpoint.project
kind: Endpoint
title: "GET /projects/:id, PUT /projects/:id, DELETE /projects/:id"
definition: 단일 프로젝트를 조회, 수정, 삭제하는 리소스 수준 엔드포인트. 삭제는 연관 데이터를 cascade로 제거하며, 진행 중인 태스크가 있으면 기본적으로 거부한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.project
    type: mutates
    note: "PUT은 프로젝트 속성을 부분 수정하고, DELETE는 프로젝트와 스코프된 모든 데이터를 영구 제거한다."
  - to: concept.task
    type: reads
    note: "DELETE 실행 전 in-flight(진행 중) 태스크 존재 여부를 확인하여 삭제 허용 여부를 결정한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

경로 파라미터 `:id`로 특정 프로젝트를 지정하여 세 가지 작업을 수행한다. GET은 단일 프로젝트의 상세 정보를 반환하고, PUT은 변경이 필요한 필드만 골라 수정하며(PATCH 방식), DELETE는 해당 프로젝트와 그 아래 속한 모든 데이터를 한꺼번에 제거한다. 삭제는 되돌릴 수 없으므로 진행 중인 태스크가 있으면 기본적으로 차단하고 명시적 승인이 필요하다.

## 요청 / 응답

### GET /projects/:id — 프로젝트 단건 조회

경로 파라미터 외 추가 입력 없음. 성공 시 HTTP 200과 함께 해당 프로젝트의 전체 속성(id, key, name, slug, description, enabled, cwds, wiki_paths, 생성/수정 시각)을 반환한다. 존재하지 않는 id이면 404를 반환한다.

### PUT /projects/:id — 프로젝트 수정

변경할 필드만 포함한 JSON 바디를 전송한다. 보내지 않은 필드는 기존 값이 그대로 유지된다. 수정 가능한 필드는 다음과 같다.

- **name** — 프로젝트 표시 이름.
- **description** — 설명 텍스트.
- **enabled** — 프로젝트 활성화 여부(불리언). false로 설정하면 대시보드에서 숨겨지거나 기능이 제한될 수 있다.
- **wiki_paths** — 문서 디렉터리 경로 목록. 기존 목록을 이 값으로 전체 교체한다.

성공 시 HTTP 200과 수정된 프로젝트 전체 객체를 반환한다.

### DELETE /projects/:id — 프로젝트 삭제

해당 프로젝트와 그 아래 등록된 모든 데이터(플랜, 시나리오, 요구사항, 태스크, 라운드, 결정, 지식 항목 등)를 cascade로 영구 삭제한다.

**삭제 안전장치**: 진행 중인 태스크(in_progress 상태)가 하나라도 있으면 기본적으로 삭제를 거부하고 409를 반환한다. 의도를 명확히 하기 위해 `?force=true` 쿼리 파라미터를 추가해야만 in-flight 태스크가 있어도 강제 삭제가 진행된다.

성공 시 HTTP 204(내용 없음)를 반환한다.

## 권한 / 제약

인증 불필요. PUT은 멱등하다 — 같은 값을 반복 전송해도 결과가 동일하다. DELETE는 비가역적이며 `?force=true` 없이 진행 중 태스크가 있으면 반드시 실패한다. `:id` 경로 파라미터는 프로젝트 ULID 또는 key 중 어느 것을 지원하는지 구현 확인 필요.

## provenance

- 대시보드 SPA의 프로젝트 설정 화면이 GET과 PUT을 사용하여 정보를 표시하고 편집한다.
- CLI `sdi project update` 및 `sdi project delete` 명령이 각각 PUT과 DELETE를 호출한다.
- 삭제 전 in-flight 태스크 확인 로직은 `concept.task` 데이터를 읽어 결정한다.
- 라우터 구현 위치: `sdi-plugin/crates/daemon/src/router/project.rs`.

## 미확정 (OPEN)

- `:id` 경로 파라미터로 ULID와 key(예: `"SDI"`) 둘 다 지원하는지, ULID만 지원하는지 미확인.
- `enabled: false`로 설정했을 때의 구체적인 동작 범위(대시보드 숨김만인지, 새 작업 차단까지인지) 미결.
- `wiki_paths` 수정 시 기존 목록 전체 교체 vs. 항목 추가/제거 방식 중 어느 것을 채택할지 최종 결정 미확인.
- `?force=true` 삭제 시 in-flight 태스크를 cancelled로 자동 전환하는지, 그냥 강제 제거하는지 정책 미확인.
- cascade 삭제 범위에 사용량 로그(usage records)가 포함되는지 여부 미확인.
