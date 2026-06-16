---
id: endpoint.question
kind: Endpoint
title: "GET /questions/:id"
definition: 단일 Q&A 질문을 식별자로 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
relatesTo:
  - to: concept.plan
    type: reads
    note: "질문이 귀속된 플랜 정보가 응답에 포함된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

질문 식별자를 경로 파라미터로 받아 해당 Q&A 질문의 전체 정보를 반환한다. 질문 내용뿐만 아니라 답변이 등록된 경우 답변 내용과 답변자, 답변 시각도 함께 포함한다.

## 요청 / 응답

**GET /questions/:id**

경로 파라미터:

- `id` — 조회할 질문의 식별자. 필수.

응답 필드:

- `id` — 질문 식별자.
- `plan_id` — 질문이 속한 플랜 식별자.
- `body` — 질문 내용 텍스트.
- `asker` — 질문 작성자. 기본값은 `"agent"`.
- `status` — 현재 상태. `open` 또는 `answered`.
- `answer` — 답변 텍스트. 아직 답변이 없으면 null.
- `answered_by` — 답변 작성자. 아직 답변이 없으면 null.
- `answered_at` — 답변이 등록된 시각. 아직 답변이 없으면 null.

질문이 존재하지 않으면 404 를 반환한다.

## 권한 / 제약

- 읽기 전용 엔드포인트다. 상태 변경이 없다.
- `answer`, `answered_by`, `answered_at` 은 `POST /questions/:id/answer` 가 호출된 이후에만 값이 채워진다.

## provenance

Q&A 질문 단건 상세 조회는 대시보드에서 미결 질문의 전체 맥락(질문 내용, 답변 여부, 답변 내용)을 확인하는 용도로 설계되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/collab.rs` 에 위치한다.

## 미확정 (OPEN)

- 응답에 `plan_id` 를 직접 포함하는지, 또는 플랜 객체를 인라인으로 포함하는지 확인이 필요하다.
- `asker` 와 `answered_by` 가 자유 문자열인지 시스템이 관리하는 식별자인지 명시되지 않았다.
- 질문을 수정(`PATCH`)하거나 삭제(`DELETE`)하는 엔드포인트가 별도로 존재하는지 확인이 필요하다.
