---
id: endpoint.comment
kind: Endpoint
title: "GET /comments/:id, PATCH /comments/:id, DELETE /comments/:id"
definition: 단일 댓글을 식별자로 조회, 수정, 삭제한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
relatesTo:
  - to: concept.task
    type: reads
    note: "댓글이 태스크에 연결되어 있을 수 있으며, 태스크 컨텍스트 조회 시 함께 노출된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

댓글 식별자를 경로 파라미터로 받아 해당 댓글을 조회하거나, 내용을 수정하거나, 삭제한다. 세 가지 메서드가 동일한 경로를 공유한다.

## 요청 / 응답

**GET /comments/:id — 단일 댓글 조회**

경로 파라미터:

- `id` — 조회할 댓글의 식별자. 필수.

응답: 댓글 식별자, 본문 텍스트, 작성자, 앵커 엔티티 정보(앵커 종류와 앵커 식별자), 생성 시각, 마지막 수정 시각.

댓글이 존재하지 않으면 404 를 반환한다.

**PATCH /comments/:id — 댓글 수정**

경로 파라미터:

- `id` — 수정할 댓글의 식별자. 필수.

요청 본문:

- `body` — 새로운 댓글 내용. 필수.

응답: 수정된 댓글 객체. `comment.updated` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

**DELETE /comments/:id — 댓글 삭제**

경로 파라미터:

- `id` — 삭제할 댓글의 식별자. 필수.

성공 응답: 204 No Content. `comment.deleted` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

## 권한 / 제약

- 세 메서드 모두 지정한 댓글이 존재하지 않으면 404 를 반환한다.
- `PATCH` 에서 `body` 가 없거나 빈 문자열이면 400 오류를 반환한다.
- 댓글 작성자 검증(작성자 본인 외 수정·삭제 불가 등) 정책이 구현되었는지 확인이 필요하다.

## provenance

댓글 단건 관리 기능은 협업 흐름의 기본 CRUD 요건에서 도출되었다. SSE 이벤트(`comment.updated`, `comment.deleted`)는 실시간 대시보드 동기화를 위해 설계되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/collab.rs` 에 위치한다.

## 미확정 (OPEN)

- `PATCH` 가 부분 수정(본문만 변경 가능)인지, 전체 교체인지 명확히 확인이 필요하다.
- 삭제된 댓글을 실제로 데이터베이스에서 제거하는 hard delete 인지, 삭제 플래그를 세우는 soft delete 인지 확인되지 않았다.
- SSE 이벤트 페이로드에 앵커 엔티티 정보가 포함되는지 여부가 명시되지 않았다.
- 작성자 기반 수정·삭제 권한 제어가 현재 구현에 포함되어 있는지 확인이 필요하다.
