---
id: endpoint.relations-delete
kind: Endpoint
title: "DELETE /relations/:id"
definition: 관계 ID로 태스크 간 관계를 삭제한다. 삭제 성공 시 task.relation.deleted SSE 이벤트가 발행된다.
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

태스크 간에 설정된 관계를 관계 자체의 ID(`:id`)를 사용해 삭제한다. 관계는 태스크 ID가 아닌 관계 고유 ID로 식별된다. 삭제가 완료되면 `task.relation.deleted` SSE 이벤트가 발행되어 대시보드 등 구독자에게 실시간으로 알린다.

이 엔드포인트는 태스크 ID가 아니라 관계 ID를 경로로 받는다는 점에서 `/tasks/:id/relations` 와 구분된다. 삭제 대상 관계의 ID는 GET `/tasks/:id/relations` 응답에서 얻을 수 있다.

## 요청 / 응답

**요청**
- 메서드: DELETE
- 경로 파라미터: `id` — 삭제할 관계의 고유 ID (태스크 ID가 아님)
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**응답**
- 성공(200 또는 204): 관계가 삭제되었음을 나타낸다. SSE 채널에 `task.relation.deleted` 이벤트 발행.
- 관계 없음(404): 지정한 관계 ID가 존재하지 않을 때 반환된다.

## 권한 / 제약

- 삭제 성공 시 SSE 이벤트(`task.relation.deleted`)가 발행되어 실시간 구독자에게 알린다.
- 부모-자식 관계(`kind=parent`)도 이 엔드포인트로 삭제 가능한지, 아니면 별도 제약이 있는지 미확인이다.
- 이미 삭제된 관계 ID로 재시도했을 때의 응답(멱등 vs 404) 미확인.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- POST `/tasks/:id/relations` 로 생성된 관계의 삭제 경로다. 관계 생성 응답에서 받은 관계 ID를 사용해 이 엔드포인트를 호출한다.

## 미확정 (OPEN)

- 성공 응답 코드가 200(본문 있음)인지 204(본문 없음)인지 구현 확인 전까지 미검증 상태다.
- 부모-자식 관계 삭제 가능 여부 및 그 경우 태스크 계층에 미치는 영향 미확인.
- SSE 이벤트 페이로드에 삭제된 관계의 양쪽 태스크 ID가 포함되는지 미확인.
