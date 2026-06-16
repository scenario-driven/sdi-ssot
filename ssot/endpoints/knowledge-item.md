---
id: endpoint.knowledge-item
kind: Endpoint
title: "GET /knowledge/:id, PUT /knowledge/:id, DELETE /knowledge/:id"
definition: 단일 지식 항목을 조회하거나, 내용을 수정하거나, 삭제한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/knowledge.rs"
relatesTo:
  - to: concept.knowledge-entry
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

하나의 지식 항목(knowledge entry)에 대한 개별 조회, 수정, 삭제를 담당하는 엔드포인트 집합이다. 세 메서드 모두 경로 파라미터 `:id`로 대상 항목을 특정한다.

`GET /knowledge/:id`는 항목의 전체 내용을 반환한다.

`PUT /knowledge/:id`는 항목의 제목(`title`), 본문(`body`), 태그(`tags`)를 갱신한다. 갱신은 전달한 필드만 변경되는 부분 업데이트 방식인지, 전체 교체 방식인지는 구현에 따라 결정된다.

`DELETE /knowledge/:id`는 항목을 데이터베이스에서 제거하고 `knowledge.deleted` SSE 이벤트를 발행한다.

## 요청 / 응답

**공통 경로 파라미터**

- `:id` — 대상 지식 항목의 식별자.

**GET /knowledge/:id**

요청 본문 없음. 성공 시 해당 항목의 전체 필드(id, project_id, scope, kind, title, body, tags, source_ref, 생성일시 등)를 반환한다.

**PUT /knowledge/:id 요청 본문 (JSON)**

- `title` (선택) — 변경할 제목.
- `body` (선택) — 변경할 본문 내용.
- `tags` (선택) — 변경할 태그 목록.

성공 시 갱신된 항목 전체를 반환한다.

**DELETE /knowledge/:id**

요청 본문 없음. 성공 시 삭제 확인 응답을 반환하고, `knowledge.deleted` SSE 이벤트를 발행한다.

## 권한 / 제약

- 존재하지 않는 `:id`에 대한 요청은 404를 반환한다.
- DELETE는 복구할 수 없는 영구 삭제다. 삭제된 항목을 참조하는 외래키 관계가 있을 경우의 처리 방식은 구현에 따른다.
- PUT에서 `scope`나 `kind` 변경이 허용되는지는 미확인이다.

## provenance

`POST /knowledge, GET /knowledge`(endpoint.knowledge)에서 생성된 항목을 개별 단위로 관리하기 위한 파트너 엔드포인트다. `sdi-plugin/crates/daemon/src/router/knowledge.rs`에 함께 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- PUT이 전체 교체(replace) 방식인지 부분 업데이트(patch) 방식인지 미확인이다.
- PUT에서 `scope`와 `kind` 필드의 변경 허용 여부가 명시되지 않았다.
- DELETE 시 해당 항목을 `source_ref`로 참조하는 다른 엔티티에 미치는 영향이 명시되지 않았다.
- `knowledge.deleted` SSE 이벤트의 페이로드 구조(삭제된 id만 포함하는지, 전체 항목 스냅샷을 포함하는지)가 미확인이다.
