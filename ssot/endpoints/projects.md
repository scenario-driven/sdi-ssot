---
id: endpoint.projects
kind: Endpoint
title: "POST /projects, GET /projects"
definition: 프로젝트를 생성하거나 전체 프로젝트 목록을 조회하는 컬렉션 수준 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.project
    type: mutates
    note: "POST는 새 프로젝트를 생성(삽입)하고, GET은 전체 프로젝트를 읽어 반환한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

SDI에서 모든 작업은 프로젝트 단위로 범위가 구분된다. 이 엔드포인트는 프로젝트를 처음 등록하거나(POST), 현재 등록된 프로젝트 전체를 확인하는(GET) 두 가지 역할을 담당한다. `sdi init` 같은 CLI 명령이 내부적으로 POST를 호출하여 프로젝트를 만든다.

## 요청 / 응답

### POST /projects — 프로젝트 생성

**요청 바디** (JSON):

- **key** (필수) — 프로젝트를 식별하는 짧은 대문자 코드 (예: `"SDI"`, `"GM"`). 전역 고유값이어야 한다.
- **name** (필수) — 사람이 읽는 프로젝트 이름.
- **slug** (선택) — URL-safe 식별자. 지정하지 않으면 key를 기반으로 자동 생성된다.
- **cwds** (선택) — 이 프로젝트와 연결할 로컬 디렉터리 경로 목록. 대시보드에서 `cwd`로 프로젝트를 조회할 때 사용된다.
- **description** (선택) — 프로젝트 설명 자유 텍스트.
- **wiki_paths** (선택) — 프로젝트 문서 디렉터리 경로 목록. 기본값은 `["docs"]`.

성공 시 HTTP 201과 함께 생성된 프로젝트 객체(id 포함)를 반환한다. key가 이미 존재하면 409(중복 충돌)를 반환한다.

### GET /projects — 프로젝트 목록 조회

파라미터 없음. 등록된 모든 프로젝트를 배열로 반환한다. 각 항목에는 id, key, name, slug, 활성화 여부, 연결된 cwd 목록, 생성 시각이 포함된다. 프로젝트가 하나도 없으면 빈 배열을 반환한다.

## 권한 / 제약

인증 불필요. POST는 데이터를 변경하므로 멱등하지 않다 — 같은 key로 두 번 호출하면 두 번째는 실패한다. GET은 읽기 전용이며 사이드이펙트 없다.

## provenance

- `sdi init` CLI 명령이 POST를 호출하여 프로젝트를 초기화한다.
- 대시보드 SPA의 프로젝트 선택 화면이 GET을 호출하여 목록을 표시한다.
- 라우터 구현 위치: `sdi-plugin/crates/daemon/src/router/project.rs`.

## 미확정 (OPEN)

- GET 응답에 페이지네이션(limit/offset/cursor)이 필요한지 또는 전체 반환으로 충분한지 미결.
- `cwds` 필드에 동일한 경로가 여러 프로젝트에 중복 등록되는 경우의 해소 정책 미확인.
- `wiki_paths` 기본값 `["docs"]`가 생성 시 자동 삽입되는지 아니면 명시적으로 보내야 하는지 미확인.
- key 형식 검증 규칙(대소문자, 길이 제한, 허용 문자) 공식 명세 없음.
