---
id: endpoint.autonomy-policies-circuit-breaker
kind: Endpoint
title: "POST /autonomy_policies/circuit_breaker"
definition: >
  D18 긴급 회로 차단기. 프로젝트의 모든 자율성 정책을 즉시 L3으로 강등시키는 비상 조치 엔드포인트.
  단일 버튼 액션으로 에이전트의 자율 행동을 제한하기 위해 설계되었으며, 활성 인시던트 중 사람이 개입해 통제권을 되찾는 수단이다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/autonomy_policy.rs"
relatesTo:
  - to: concept.autonomy-policy
    type: mutates
  - to: concept.circuit-breaker
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

D18 설계 결정에 따른 비상 회로 차단기 엔드포인트다. 이 엔드포인트를 호출하면 해당 프로젝트에 등록된 모든 자율성 정책이 즉시 L3 수준으로 일괄 강등된다. 개별 정책을 하나씩 수정할 필요 없이 단 한 번의 호출로 에이전트의 자율 행동 범위를 축소한다. UI의 패닉 버튼이나 사람의 직접 개입 시나리오를 위해 설계되었다.

정책이 변경된 직후 `circuit_breaker.triggered` SSE 이벤트가 발행되어 모든 구독자에게 즉시 알린다.

## 요청 / 응답

**POST /autonomy_policies/circuit_breaker**

요청 본문:

- `project_id` (필수): 회로 차단을 적용할 프로젝트 식별자.
- `actor` (선택, 기본값 `"agent"`): 이 조치를 실행한 주체. 사람이 직접 호출한 경우 사람 식별자를 기록.
- `reason` (필수, 비어 있으면 안 됨): 회로 차단을 실행한 이유. 감사 추적과 사후 검토를 위해 반드시 기술해야 한다.

응답: 강등된 정책 행의 수(count).

## 권한 / 제약

- `reason` 필드는 비어 있을 수 없다. 이유 없는 비상 조치는 허용하지 않는다.
- 이 엔드포인트는 파괴적이다. 호출 즉시 프로젝트 전체의 자율성이 L3으로 낮아진다. 원상복구는 개별 정책을 다시 POST해야 한다.
- `circuit_breaker.triggered` SSE 이벤트가 발행되므로, 이를 구독하는 오케스트레이터나 에이전트는 즉시 동작 모드를 바꿔야 한다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/autonomy_policy.rs`에서 유추됨. D18(긴급 회로 차단기) 설계 결정 참조.

## 미확정 (OPEN)

- 강등 후 원상복구(L3 이상으로 다시 올리기) 절차가 별도로 정의되어 있는지 미확인.
- L3 강등 시 이미 진행 중인 에이전트 작업의 처리 방식(중단 vs. 완료 후 차단) 미확인.
- `actor`가 사람임을 검증하는 인증 메커니즘 여부 미확인.
- 강등 대상에 `forced=true` 정책도 포함되는지 미확인.
