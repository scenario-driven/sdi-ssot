---
id: endpoint.decisions-status
kind: Endpoint
title: "POST /decisions/:id/status"
definition: 특정 결정의 상태를 변경한다. 주로 롤백 이후 원래 결정을 superseded로 표시하거나, 이견(dissensus) 이후 제안을 rejected로 공식 처리하는 데 사용된다. 상태 변경 후 decision.status-changed SSE를 발행한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
relatesTo:
  - to: concept.decision
    type: mutates
    note: "결정의 status 필드를 변경한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

결정 레코드의 상태를 변경하는 전용 엔드포인트다. 결정은 일반적으로 생성 시 `accepted` 상태로 시작하지만, 이후 협상 흐름이나 번복 절차에 따라 상태가 달라질 수 있다.

주요 사용 시나리오는 두 가지다:

**롤백 후 대체 표시**: D28 번복 절차가 실행된 후, 원래 결정을 `superseded` 상태로 전환하여 더 이상 유효하지 않음을 명시한다. 새 롤백 결정(`reversal_of`로 원본을 참조)이 생성된 뒤, 이 엔드포인트로 원본을 대체 처리하는 것이 표준 흐름이다.

**이견 후 거부 처리**: dissensus 이후 제안을 계속 진행하지 않기로 결정한 경우, 해당 제안 결정을 `rejected`로 공식 전환하여 협상이 종료되었음을 기록한다.

상태 변경 성공 후에는 `decision.status-changed` SSE 이벤트가 발행되어 대시보드나 다른 에이전트가 변경 사실을 실시간으로 수신할 수 있다.

## 요청 / 응답

**요청**

- 경로 파라미터: `:id` — 상태를 변경할 결정의 ULID
- 바디: 변경할 목표 상태(status) 문자열 (예: `superseded`, `rejected`, `accepted`)

**응답**

- 성공 시 변경 후의 결정 레코드를 반환한다.
- 변경 직후 `decision.status-changed` SSE가 발행된다.
- 존재하지 않는 ID는 404로 응답한다.

## 권한 / 제약

- 허용되지 않는 상태 전환이 있는지, 아니면 임의의 상태값으로 자유롭게 변경 가능한지 미확인.
- `superseded` 상태로 전환할 때 대체한 새 결정의 ID를 함께 제공해야 하는지 여부 미확인.

## provenance

SDI 플러그인 데몬의 결정 라우터(`sdi-plugin/crates/daemon/src/router/decision.rs`)에 구현되어 있다. `endpoint.decisions`(생성), `endpoint.decision`(단일 조회)와 함께 결정 도메인 인터페이스를 구성한다. D28 번복 절차와 M3 협상 흐름의 생애주기 관리를 지원하기 위해 설계되었다.

## 미확정 (OPEN)

- 허용 가능한 상태 전환 매트릭스(예: accepted→rejected 가능 여부)가 도메인 계층에서 강제되는지 미확인.
- `decision.status-changed` SSE 이벤트의 페이로드 형식 미확인.
- 이미 `rejected` 또는 `superseded`인 결정을 다시 `accepted`로 되돌리는 것이 허용되는지 미확인.
- 동일 상태로의 전환(no-op)을 허용하는지 아니면 오류로 처리하는지 미확인.
