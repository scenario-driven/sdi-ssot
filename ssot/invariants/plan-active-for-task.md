---
id: invariant.plan-active-for-task
kind: Invariant
title: Round를 시작하려면 Plan이 active 상태여야 한다
definition: "Round를 생성하거나 active로 전환하려면 해당 Plan이 이미 approve되어 active 상태여야 한다. Plan이 draft이거나 존재하지 않으면 Round를 시작할 수 없다. Plan approve 게이트는 확인된 시나리오 1개 이상 + 모든 GWT 유효성 통과를 요구한다."
governs:
  - concept.plan
  - concept.round
  - concept.scenario
  - domain.planning
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
  - "sdi-plugin/crates/daemon/src/router/round.rs"
  - "sdi-plugin/crates/core/src/plan.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 작업 흐름은 Plan → Round → Task의 계층 구조를 따른다. Round는 Plan의 승인된 시나리오를 기반으로 LLM이 Task를 분해하는 실행 단위다. 따라서 Plan이 승인(active 전환)되지 않은 상태에서는 어떤 시나리오를 기반으로 Round를 진행할지 정의되어 있지 않다.

Plan approve 게이트(D8)가 두 조건을 검사한다: (1) confirmed 상태의 시나리오가 최소 1개 있어야 한다, (2) 모든 시나리오의 GWT 세 필드가 유효해야 한다. Task 개수는 이 게이트의 조건이 아니다. Task는 LLM이 Round 시작 시점에 런타임에 생성하므로(D3) Plan approve 시점에 존재하지 않아도 된다.

이 불변식은 "빈 Plan에서 Round를 시작하는" 실수를 방지하고, 사람이 시나리오를 검토·승인한 후에만 LLM이 자율 실행 단계에 진입하도록 보장하는 유일한 인간 개입 게이트다.

## 깨지면 무슨 일이 일어나나

draft Plan에서 Round를 시작할 수 있다면 LLM이 시나리오 없이 Task를 만들거나, 사람이 아직 검토하지 않은 미완성 시나리오를 기반으로 구현을 시작한다. GWT가 비어있는 시나리오를 기반으로 분해된 Task는 무엇을 구현해야 하는지 정의가 없어 LLM이 임의로 판단해야 한다. 또한 사람의 승인 없이 자율 실행이 시작되어 D16의 "act with policy, not default ask" 철학이 의도하는 "정의된 정책 아래 자율 실행"이 아니라 "무정의 상태에서의 자율 실행"이 된다.

## 코드에서 어떻게 강제되나

Plan approve 엔드포인트(`crates/daemon/src/router/plan.rs`)는 `PATCH /plans/:id/approve` 요청 시 confirmed 시나리오 수와 GWT 유효성을 확인하며, 조건 불충족 시 `SCENARIOS_REQUIRED` 오류를 반환한다. Round 생성 엔드포인트(`crates/daemon/src/router/round.rs`)는 해당 Plan의 lifecycle이 `active`인지 확인하며, draft이면 `PLAN_NOT_ACTIVE` 오류를 반환한다. HOOK_ENFORCEMENT.md #2 인수기준으로 명시되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D8 Plan approve 게이트 결정 엔트리) 연결 필요
