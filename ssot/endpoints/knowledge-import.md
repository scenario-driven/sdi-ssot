---
id: endpoint.knowledge-import
kind: Endpoint
title: "POST /knowledge/import"
definition: >
  지식 항목(knowledge entry) 배열을 받아 프로젝트에 일괄 삽입한다.
  이미 존재하는 항목은 조용히 건너뛰는 최선-노력(best-effort) 업서트 방식으로 동작한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/aggregate.rs"
relatesTo:
  - to: concept.knowledge-entry
    type: mutates
    note: 지식 항목을 새로 삽입하거나 이미 있으면 건너뛴다
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`GET /knowledge/export` 로 내보낸 지식 항목 묶음을 다시 가져오는 엔드포인트다. 반대 방향 작업이므로 export 와 쌍을 이룬다. 항목 식별자(id) 기준으로 이미 저장된 항목은 덮어쓰지 않고 조용히 건너뛰기 때문에, 같은 번들을 여러 번 가져와도 중복이 생기지 않는다. 가져오기에 성공한 항목 수를 응답으로 돌려준다.

## 요청 / 응답

**요청**

- 메서드: `POST`
- 경로: `/knowledge/import`
- 본문 (JSON)
  - `project_id` (선택): 지식 항목을 삽입할 프로젝트 식별자. 생략하면 각 지식 항목이 원래 갖고 있던 project_id 를 사용한다.
  - `knowledge` (필수): 삽입할 지식 항목 객체의 배열. `GET /knowledge/export` 가 반환한 배열과 동일한 형식을 기대한다.

**응답**

- 성공 시 HTTP 200
- 본문: `{ "imported": <숫자> }` 형태의 JSON. 숫자는 실제로 새로 삽입된 항목 수이며, 건너뛴 항목은 포함되지 않는다.

## 권한 / 제약

- `knowledge` 배열이 비어 있으면 삽입 없이 `{ "imported": 0 }` 을 반환한다.
- 이미 존재하는 항목(같은 id)은 업데이트하지 않고 건너뛴다(silent skip). 기존 데이터를 덮어쓰려면 별도 수단이 필요하다.
- 데몬이 실행 중이어야 요청을 처리할 수 있다.

## provenance

- 라우트 출처: `sdi-plugin/crates/daemon/src/router/aggregate.rs`
- `GET /knowledge/export` 와 쌍을 이루는 역방향 엔드포인트다.

## 미확정 (OPEN)

- `project_id` 를 본문에서 재정의했을 때 개별 항목의 원래 project_id 와 충돌 처리 방식이 명시되지 않았다.
- 한 번에 처리할 수 있는 항목 수 상한이 있는지 불명확하다.
- 부분 실패(일부 항목 삽입 오류) 시 트랜잭션 처리 여부가 명시되지 않았다.
- 인증·권한 검사 여부가 명시되지 않았다.
