---
id: endpoint.run
kind: Endpoint
title: "GET /runs/:id"
definition: 단일 run 기록의 전체 내용을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.run
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 run 기록 하나를 조회하는 엔드포인트다. `POST /runs`로 생성된 run의 현재 상태, 실행 결과, 메모 등 전체 정보를 반환한다. run이 아직 진행 중이라면 `finished_at`과 `result` 필드가 비어있는 상태로 반환된다.

## 요청 / 응답

**경로 파라미터**

- `:id` — 조회할 run의 식별자.

**요청 본문**

없음.

**응답 필드**

- `task_id` — 이 run이 수행한 태스크의 식별자.
- `actor` — 실행 주체. 예: `"agent"`.
- `session_id` — 이 run이 속한 세션 식별자. 세션이 없으면 null.
- `started_at` — run이 시작된 타임스탬프.
- `finished_at` — run이 종료된 타임스탬프. 아직 진행 중이면 null.
- `result` — 종료 결과. `passed`, `failed`, `error`, `cancelled` 중 하나. 아직 종료되지 않았으면 null.
- `notes` — 종료 시 기록된 메모. 없으면 null.
- `payload` — run 시작 시 전달된 추가 데이터. 없으면 null.

## 권한 / 제약

- 읽기 전용이다. 데이터를 변경하지 않는다.
- 존재하지 않는 `:id`에 대한 요청은 404를 반환한다.

## provenance

`POST /runs`(endpoint.runs)로 시작된 run의 상세 내용을 조회하기 위한 파트너 엔드포인트다. `POST /runs/:id/finish`(endpoint.runs-finish)로 종료된 이후에는 `result`와 `finished_at`이 채워진 상태로 반환된다. `sdi-plugin/crates/daemon/src/router/run.rs`에 함께 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- `payload` 필드의 구체적인 데이터 구조가 정의되어 있는지 미확인이다.
- `notes` 필드가 단일 문자열인지 구조화된 형식(예: JSON 배열)인지 미확인이다.
- run 상태(`in_progress`, `finished` 등)를 명시적으로 나타내는 별도 상태 필드가 있는지 미확인이다.
