---
id: endpoint.scenarios-confirm
kind: Endpoint
title: "POST /scenarios/:id/confirm"
definition: >
  시나리오를 draft(초안) 상태에서 confirmed(확정) 상태로 전환하는 엔드포인트.
  확정된 시나리오는 D8 플랜 승인 게이트의 조건(최소 1개 confirmed)을 충족시키며,
  완료 시 SSE 이벤트를 발행한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario
    type: mutates
  - to: concept.plan
    type: relates-to
    note: "confirmed count feeds D8 approve gate"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 시나리오를 초안(draft) 상태에서 확정(confirmed) 상태로 전환한다. 확정은 해당 시나리오가 검토를 거쳐 구현 기준으로 삼을 준비가 되었음을 의미한다.

SDI의 D8 플랜 승인 게이트는 플랜 내에 확정된 시나리오가 최소 1개 이상 존재해야 통과할 수 있다. 이 엔드포인트가 호출되어 시나리오가 confirmed 상태가 되면 해당 조건을 충족할 수 있게 된다.

요청 본문은 필요 없다. 경로의 `:id`만으로 대상 시나리오를 특정한다. 상태 전환 완료 시 `scenario.confirmed` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

- 메서드: POST
- 경로 파라미터: `:id` (시나리오 고유 식별자)
- 요청 본문: 없음
- 응답: 상태가 confirmed로 갱신된 시나리오 전체 필드
- 이미 confirmed 상태인 시나리오에 다시 호출하면 멱등적으로 처리되거나 409를 반환할 수 있음(미확인)
- 전환 완료 즉시 `scenario.confirmed` SSE 이벤트 발행

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- retired 상태인 시나리오를 confirm 하는 것이 허용되는지는 미확인이다.
- 이 엔드포인트는 상태를 confirmed로 전환하는 단방향 동작이다. confirmed → draft 역전환은 별도 엔드포인트를 통해 이루어지는지, 아니면 불가능한지 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `POST /scenarios/:id/confirm` 핸들러를 추론하여 작성하였다. D8 플랜 승인 게이트와의 연결은 SDI PRD(scenario-engine-prd.md) 및 SSOT 플랜 개념 노드에서 참조하였다. SSE 이벤트 이름 `scenario.confirmed` 는 데몬 이벤트 버스 패턴에서 추론한 값이다.

## 미확정 (OPEN)

- confirmed 상태에서 다시 POST를 호출했을 때 멱등적으로 200을 반환하는지, 아니면 409 Conflict를 반환하는지 확인 필요.
- confirmed → draft 역전환이 필요한 경우 어떤 경로(엔드포인트 또는 PUT)를 사용하는지 미확인.
- retired 시나리오를 confirm 시도했을 때의 동작(허용/거부) 미확인.
- `scenario.confirmed` SSE 페이로드 구조(전체 시나리오 포함 여부, 플랜 내 confirmed count 포함 여부) 미확인.
