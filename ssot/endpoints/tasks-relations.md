---
id: endpoint.tasks-relations
kind: Endpoint
title: "POST /tasks/:id/relations, GET /tasks/:id/relations"
definition: 태스크 간 비부모 관계를 추가(POST)하거나 특정 태스크의 모든 관계를 조회(GET)한다. 부모-자식 관계는 이 엔드포인트로 생성할 수 없으며, 반드시 /tasks/:id/decompose 를 사용해야 한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.task
    type: mutates
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

하나의 경로(`:id`)에 두 가지 메서드를 제공하는 복합 엔드포인트다.

**POST**: 지정한 태스크와 다른 태스크 사이에 비부모 관계를 생성한다. 관계 종류(kind)로 `related` 등의 비부모 유형만 허용한다. `kind=parent` 를 요청하면 거부된다 — 부모-자식 관계는 `/tasks/:id/decompose` 경로를 통해서만 만들 수 있다. 관계가 성공적으로 생성되면 `task.relation.created` SSE 이벤트가 발행된다.

**GET**: 지정한 태스크에 연결된 모든 관계 목록을 반환한다. 방향(해당 태스크가 출발지인지 도착지인지)과 관계 종류를 포함한다.

## 요청 / 응답

**POST 요청**
- 경로 파라미터: `id` — 관계를 추가할 기준 태스크의 ID
- 요청 본문(JSON): 관계 대상 태스크 ID, 관계 종류(kind), 선택적 설명
- 제약: `kind=parent` 는 400 오류로 거부됨

**POST 응답**
- 성공(201): 생성된 관계 정보 반환. SSE 채널에 `task.relation.created` 이벤트 발행.
- 잘못된 kind(400): `kind=parent` 를 포함한 요청은 거부된다.
- 태스크 없음(404): 기준 태스크 또는 대상 태스크가 존재하지 않을 때.

**GET 요청**
- 경로 파라미터: `id` — 관계를 조회할 태스크의 ID
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**GET 응답**
- 성공(200): 해당 태스크의 모든 관계 배열. 각 항목은 관계 ID, 상대 태스크 ID, 관계 종류, 방향 등을 포함한다.
- 태스크 없음(404): 지정한 태스크가 존재하지 않을 때.

## 권한 / 제약

- `kind=parent` 생성 시도는 명시적으로 400 오류로 차단된다. 부모 관계 생성은 `/tasks/:id/decompose` 가 전담한다.
- POST 성공 시 SSE 이벤트(`task.relation.created`)가 발행되어 대시보드 등 구독자에게 실시간으로 알린다.
- 이미 존재하는 동일 방향·동일 종류의 관계를 중복 생성했을 때의 처리 방식은 미확인이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- 부모-자식 관계와 비부모 관계를 분리한 설계는 태스크 분해(decompose) 의미를 명시적으로 보호하기 위한 의도적 제약이다.

## 미확정 (OPEN)

- 허용되는 `kind` 값의 전체 목록(예: `related`, `blocks`, `duplicates` 등)이 구현 확인 전까지 미검증 상태다.
- GET 응답에서 방향(outgoing/incoming)을 어떻게 구분해 표현하는지 미확인.
- 중복 관계 생성 시 409 오류를 반환하는지, 멱등하게 처리하는지 미확인.
- SSE 이벤트 페이로드의 정확한 구조 미확인.
