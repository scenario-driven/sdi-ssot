---
id: invariant.cycle-required-for-task
kind: Invariant
title: Task는 반드시 active인 Round에 귀속되어야 한다
definition: "Task는 active 상태의 Round에 귀속된 runtime 산출물로서만 존재할 수 있다. active Round 없이 Task를 생성하거나, active가 아닌 Round에 Task를 생성하는 것은 허용되지 않는다."
governs:
  - concept.task
  - concept.round
  - concept.run
  - domain.task-decomposition
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
  - "sdi-plugin/crates/core/src/round.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

Task는 SDI에서 LLM이 런타임에 시나리오를 기반으로 생성하는 실행 단위다(D3). Task는 항상 특정 Round에 귀속되며, 그 Round는 active 상태여야 한다. Round가 planning 또는 completed 상태이거나 아예 존재하지 않으면 Task를 생성할 수 없다.

이 불변식의 이유는 Task evidence의 집계 구조에 있다. Task가 done이 될 때 제출하는 증거(`invariant.evidence-required-on-done`)는 귀속된 Round의 `scenario_results`에 기입된다. Round가 active가 아니라면 scenario_results에 기입하는 행위 자체가 의미를 잃는다. 완료된 라운드에 Task를 생성하면 이미 확정된 회귀 기준이 사후에 오염된다.

Task는 사람이 직접 만들지 않으며(D3), LLM의 scenario-decomposer specialist가 시나리오를 분해하여 생성한다. 이 분해는 Round가 active로 전환된 시점에 시작된다.

## 깨지면 무슨 일이 일어나나

active Round 없이 Task를 만들 수 있다면, 어떤 회귀 기준에 대한 작업인지 맥락이 없는 Task가 생성된다. 그 Task가 done이 되어도 scenario_results에 기입할 Round가 없어 회귀 추적이 불가능해진다. completed Round에 Task를 사후 추가하면 이미 확정된 라운드 결과가 변경되어 R(N+1)의 carry_over_results가 오염된다. Task의 `parent_scenario_ids`가 있어도 어떤 라운드의 어떤 검증 회차를 위한 것인지 알 수 없다.

## 코드에서 어떻게 강제되나

데몬의 Task 생성 엔드포인트(`crates/daemon/src/router/task.rs`)는 요청의 `round_id`를 받아 해당 Round가 존재하는지, lifecycle이 `active`인지 확인한다. active가 아니면 `ROUND_NOT_ACTIVE` 오류를 반환한다. `invariant.one-active-cycle`에 의해 한 Plan에 active Round가 하나뿐이므로, Task 생성은 실질적으로 "현재 진행 중인 Round"에 자동으로 귀속된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(Task-Round 귀속 강제 정책의 결정 엔트리) 연결 필요
