---
id: invariant.scenario-id-one-to-one
kind: Invariant
title: 시나리오 식별자는 플랜 안에서 하나의 GWT 명제에 1:1 대응한다
definition: "하나의 Scenario 행(row)은 하나의 Given/When/Then 명제에만 대응하며, 여러 행동을 하나의 시나리오에 묶거나 시나리오 없이 Task를 직접 연결하는 것은 허용되지 않는다."
governs:
  - concept.scenario
  - concept.given-when-then
  - concept.scenario-id
  - domain.scenario-management
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
  - "sdi-plugin/crates/core/src/scenario.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI에서 시나리오는 Given(전제 상황) / When(행위) / Then(결과) 세 필드가 모두 채워진 단일 자연어 명제다. 하나의 `Scenario` 행이 여러 When 행위를 담거나, Then 결과를 복수로 나열하거나, 다른 시나리오의 내용을 포함하는 것은 허용되지 않는다. "atomic(단일 명제) 권장"은 강력한 설계 원칙이며, GWT 세 필드 각각이 비어 있을 수 없다는 규칙은 코드 수준에서 강제된다.

이 불변식의 핵심 이유는 **회귀 검증의 단위성**이다. R2+ 라운드에서 regression-runner가 이전 라운드의 passing 시나리오를 재검증할 때, 각 시나리오는 독립적으로 passing/failing/impacted 상태가 될 수 있어야 한다. 하나의 시나리오가 여러 명제를 포함하면 일부만 실패했을 때 어떤 명제가 깨진 것인지 추적이 불가능해지고, Task 분해(`invariant.evidence-required-on-done`에서 다루는 증거 연결)도 모호해진다.

시나리오 의존성(`depends_on` DAG)은 여러 시나리오 간 논리적 선후 관계를 표현하는 수단이며, 이것이 여러 명제를 하나의 시나리오에 묶는 대안이다.

## 깨지면 무슨 일이 일어나나

하나의 시나리오가 복수 명제를 담으면 회귀 검증의 단위가 무너진다. "이 시나리오가 passing이다"라는 의미가 불명확해지고, 어떤 조건이 충족되어야 passing인지 LLM마다 다르게 해석한다. Task evidence 연결(`scenario_id → result → evidence_ref`)이 어느 명제에 대한 증거인지 구분할 수 없어 R(N)의 자동 회귀 집계가 신뢰를 잃는다. 또한 `depends_on` DAG의 위상 정렬을 통한 병렬 실행 계획이 오염된다(단일 시나리오가 여러 의존성을 동시에 갖는 것처럼 보임).

## 코드에서 어떻게 강제되나

데몬의 시나리오 생성 엔드포인트(`crates/daemon/src/router/scenario.rs`)는 `given`, `when`, `then` 세 필드 각각이 비어 있으면 `GWT_EMPTY` 오류를 반환하며 저장을 거부한다. 이는 PRD D5 결정의 직접 구현이며 HOOK_ENFORCEMENT.md #1 인수기준으로도 명시되어 있다. 자유 형식 옵션은 API 수준에서 존재하지 않는다. `crates/core/src/scenario.rs`의 도메인 모델이 GWT 세 필드를 모두 필수 속성으로 정의한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D5 GWT 강제 결정 엔트리) 연결 필요
