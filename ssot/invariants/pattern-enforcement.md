---
id: invariant.pattern-enforcement
kind: Invariant
title: 모든 work entity는 produced_via_pattern_id를 가져야 하며, 가짜 패턴은 shape 게이트에서 거부된다
definition: "v0.5 이후 생성되는 모든 work entity(plan/requirement/scenario/task/decision/round)는 produced_via_pattern_id가 NOT NULL이어야 한다. CollaborationPattern의 pending→active 전이는 kind별 shape 유효성 검사를 통과해야 하며, 1-step workflow·단일 에이전트 swarm 같은 가짜 패턴은 active로 전환될 수 없다."
governs:
  - concept.collaboration-pattern
  - concept.pattern-lifecycle
  - concept.dispatch
  - domain.collaboration-patterns
  - domain.agent-coordination
  - domain.autonomy
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
  - "sdi-plugin/crates/daemon/src/router/task.rs"
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
  - "sdi-plugin/crates/daemon/src/router/round.rs"
  - "sdi-plugin/plugin/adapters/claude/pre-tool-use.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI v0.5에서 CollaborationPattern이 7번째 1등 시민 엔티티로 격상됨에 따라(D22), 모든 work entity는 자신이 어떤 협업 패턴을 통해 생성됐는지 기록해야 한다(D23). 이는 두 가지 메커니즘으로 강제된다.

**생성 시점 강제(D27a)**: 신규 work entity 생성 요청에 `produced_via_pattern_id`가 없으면 데몬이 자동으로 `kind='direct'` CollaborationPattern 행을 생성하고 해당 ID를 부여한다. `direct`는 메인 세션의 1인극 작업임을 명시하는 마커로, 자동 L3 cap과 대시보드 빨간 배지가 부여된다. legacy row(마이그레이션 이전)만 NULL을 허용한다.

**전이 시점 강제(D27b)**: CollaborationPattern이 `pending → active`로 전환될 때 kind별 shape 유효성 검사를 통과해야 한다. 각 패턴의 최소 요건은 다음과 같다: workflow는 `steps_json` 길이 2 이상, graph는 `reviewers_json`의 (name, stance) tuple distinct 2 이상, swarm은 `fan_out_json` 길이 2 이상, agents-as-tools는 `peer_registration_json` 길이 1 이상. 이 조건을 충족하지 못하면 `pending` 상태에 머물며 active 전환이 거부된다.

Graph 패턴의 sybil 차단이 중요하다. `distinct agent_id` 기준이 아니라 `(AgentSpec.name, AgentSpec.stance) tuple` 기준으로 distinctness를 판단한다. 같은 이름의 에이전트 2개를 투입해도 stance가 같으면 독립 판단을 보장하지 못하므로 1개로 계산된다.

가짜 패턴(1-step workflow, 단일 에이전트 swarm, 빈 peer agents-as-tools)은 active가 될 수 없으므로 D26의 L4/L5 unlock도 받지 못한다. `direct`의 L3 강제를 회피하는 경로가 차단된다.

## 깨지면 무슨 일이 일어나나

`produced_via_pattern_id` 추적이 없으면 어떤 에이전트 협업 패턴을 통해 어떤 작업이 만들어졌는지 사후에 알 수 없다. 패턴 효과 분석("어떤 패턴이 어떤 결과를 냈는가")이 불가능해진다. Shape 게이트가 없으면 1-step workflow를 생성하여 `direct`의 L3 cap을 우회하고 L5 자율 적용을 불법으로 획득할 수 있다. 특히 Graph 패턴의 sybil 차단이 없으면 같은 system prompt를 가진 에이전트 2개가 "consensus"를 형성했다고 주장할 수 있으며, 이는 실질적으로 단일 에이전트가 자신의 의견을 합의로 위장하는 것이다. 이 경우 L5 unlock이 정당한 다양성 없이 발생하여 잘못된 결정이 사용자 검토 없이 자동 적용된다.

## 코드에서 어떻게 강제되나

6개 work entity 생성 엔드포인트 모두가 `produced_via_pattern_id` 부재 시 자동 `direct` 행을 생성하는 로직을 포함한다. `POST /collaboration_patterns/:id/activate`는 kind별 shape validation 함수를 호출하며 미통과 시 `PATTERN_SHAPE_INVALID`를 반환한다. PreToolUse 훅(`plugin/adapters/claude/pre-tool-use.cjs`)은 D26의 런타임 패턴 무결성 게이트로, Agent/Task 도구 호출 시 active pattern을 데몬에 질의하여 kind별 규칙을 확인한다. Graph consensus Decision 적재 시 `(name, stance) distinct ≥ 2` 검사가 `decision.apply` 경로에서 이루어진다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D22 CollaborationPattern 1등 시민 / D23 pattern provenance / D26 shape gate / D27 shape & selection gate 결정 엔트리들) 연결 필요
