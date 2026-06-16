---
id: invariant.one-active-cycle
kind: Invariant
title: 한 Plan에서 active 상태의 Round는 하나뿐이다
definition: "한 Plan 안에서 lifecycle이 active인 Round는 동시에 하나만 존재할 수 있다. R(N)이 active인 동안 R(N+1)을 시작할 수 없으며, R(N)이 completed가 되어야 R(N+1)을 시작할 수 있다."
governs:
  - concept.round
  - concept.convergence
  - domain.round-execution
  - domain.planning
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
  - "sdi-plugin/crates/core/src/round.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 Round(R1, R2, …)는 "한 회차의 구현+검증 단위"다. R1은 신규 개발, R2+는 이전 시나리오 회귀 검증 + 신규 시나리오 검증이다. 이 두 흐름이 같은 엔진에서 순차적으로 진행되어야 한다(D7). 순차성은 이 불변식이 보장한다.

동시에 두 Round가 active가 되면 회귀 검증의 의미가 깨진다. R(N)이 완료(모든 시나리오 passing/failing 판정)되지 않은 상태에서 R(N+1)을 시작하면, R(N+1)이 R(N)의 어떤 passing 결과를 기준으로 회귀를 검증해야 하는지 정의할 수 없다. `carry_over_results`(이전 라운드 결과 복사) 로직이 미완성된 R(N)의 불완전한 데이터를 복사해 R(N+1)의 회귀 기준이 오염된다.

Round lifecycle 전이: `planning → active → completed`. `planning` 상태의 Round는 여러 개 생성할 수 있지만, `active`는 Plan 당 하나로 제한된다.

## 깨지면 무슨 일이 일어나나

두 Round가 동시에 active가 되면 Task가 어느 Round에 속하는지 모호해진다. Task evidence가 어느 Round의 `scenario_results`에 기입되어야 하는지 결정할 수 없고, 두 Round가 같은 시나리오에 대해 서로 다른 결과를 기록하면 어느 것이 현재 진실인지 알 수 없다. R2+의 자동 회귀 carry-over가 미완성 라운드를 기준으로 동작하여 이미 failing으로 파악된 시나리오가 신규 라운드에서 passing으로 표기되는 역설이 발생할 수 있다.

## 코드에서 어떻게 강제되나

데몬의 Round 활성화 엔드포인트(`crates/daemon/src/router/round.rs`)는 `planning → active` 전환 시 해당 Plan에 이미 active인 Round가 있는지 확인한다. 있으면 `ACTIVE_ROUND_EXISTS` 오류를 반환한다. Round 활성화 시에는 in-flight Task 처리(pause/abort/continue-on-noimpact, D10), disruption pending 검사(D9), carry_over_results 실행이 순서대로 이루어지며, 이 모든 단계가 단일 active Round의 전제 하에 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(Round active 단일성 강제 정책의 결정 엔트리) 연결 필요
