---
id: endpoint.autonomy-policies-resolve
kind: Endpoint
title: "GET /autonomy_policies/resolve"
definition: >
  주어진 컨텍스트(프로젝트, 플랜, 결정 종류, 패턴 종류)에 대해 현재 실질적으로 적용될 자율성 정책 행을 해석하여 반환하는 엔드포인트.
  등록된 정책들 중 특이성(specificity) 기준으로 가장 엄격한 정책 하나를 선별해 돌려준다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/autonomy_policy.rs"
relatesTo:
  - to: concept.autonomy-policy
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

훅(hook)과 오케스트레이터가 어떤 행동을 취하기 전에, 현재 컨텍스트에서 실제로 적용되는 자율성 수준을 파악하기 위해 호출하는 읽기 전용 해석 엔드포인트다. 여러 정책이 동시에 적용될 수 있는 경우 특이성(specificity) 기준으로 가장 좁고 엄격한 정책 하나를 선택해 반환한다. 정책 자체를 변경하지 않는다.

## 요청 / 응답

**GET /autonomy_policies/resolve**

쿼리 파라미터:

- `project_id` (필수): 해석할 컨텍스트의 프로젝트 식별자.
- `plan_id` (선택): 특정 플랜으로 범위를 좁힐 때 사용.
- `decision_kind` (선택): 특정 결정 종류의 정책을 해석할 때 지정.
- `pattern_kind` (선택): 특정 협업 패턴 종류의 정책을 해석할 때 지정.

응답: 컨텍스트에 가장 적합하게 해석된 단일 정책 행. 일치하는 정책이 없으면 시스템 기본값을 반환한다.

특이성 우선순위는 일반적으로 `plan_id + decision_kind + pattern_kind` 조합이 가장 구체적이며, `project_id`만 있는 경우가 가장 넓다. 여러 정책이 동일한 특이성을 가지면 가장 엄격한(낮은 L 레벨) 것이 선택된다.

## 권한 / 제약

- 읽기 전용. 정책을 변경하지 않으며 SSE 이벤트를 발행하지 않는다.
- 훅과 오케스트레이터가 행동 전 체크포인트로 호출하는 것이 설계 의도이므로, 응답 지연을 최소화해야 한다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/autonomy_policy.rs`에서 유추됨. 훅과 오케스트레이터의 사전 확인 패턴은 SDI 자율성 정책 설계 문서에서 참조.

## 미확정 (OPEN)

- 특이성 해석 알고리즘의 정확한 우선순위 규칙 미확인.
- 일치하는 정책이 없을 때 반환하는 시스템 기본값의 L 레벨 미확인.
- `decision_kind`와 `pattern_kind`를 동시에 지정했을 때의 교차 해석 동작 미확인.
