---
id: concept.consensus
kind: Concept
title: Consensus(합의)
definition: "M3 4단계 협상 흐름(D20)의 최종 단계. proposal과 critique를 거쳐 여러 에이전트가 도달한 합의로, kind=consensus인 Decision 행으로 기록된다. Dissensus(불합의)는 항상 인간 게이트로 에스컬레이션된다."
relatesTo:
  - { to: concept.decision, type: backed-by, note: "Consensus는 kind=consensus인 Decision 행으로 표현된다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "Graph 패턴은 distinct (name, stance) 튜플 2개 이상의 reviewer로 구성되어야 consensus가 성립한다(D26 sybil 차단)." }
  - { to: concept.autonomy-policy, type: relates-to, note: "autonomy mode(L3/L4/L5)는 consensus 결정이 어떤 인간 게이트 위치에서 적용되는지를 제어한다(D20)." }
  - { to: domain.decision-negotiation, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/decision.rs
  - sdi-plugin/crates/core/src/pattern.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Consensus(합의)는 SDI의 다중 에이전트 의사결정 흐름(M3 협상)의 결론이다. 여러 전문가 에이전트(specialist sub-agents)가 독립적인 관점에서 제안(proposal)을 비평(critique)한 후, 합의가 이루어지면 consensus Decision 행이 기록된다. 합의가 이루어지지 않으면 dissensus가 기록되고 인간 사용자에게 에스컬레이션된다.

**Consensus가 성립하기 위한 최소 조건(D20)**:

1. 이미 proposal Decision 행이 존재해야 한다.
2. 해당 proposal에 대한 critique Decision 행이 최소 1개 있어야 한다.
3. consensus 행은 proposal_id로 해당 proposal을 명시적으로 가리킨다.

**Autonomy mode와 consensus의 관계(D20)**:

- **L3(ask)**: consensus 결정이 있더라도 인간이 각 결정을 개별 확인해야 한다.
- **L4(act-with-review)**: consensus 시 자동으로 적용하되, 사용자가 적용 전에 취소할 수 있는 시간 창을 준다.
- **L5(act-and-notify)**: consensus 시 자동 적용하고 사용자에게 사후 통보한다. 이 모드는 D28의 reversal_plan + blast_radius_score 조건도 충족해야 한다.

**단일 에이전트 제약(D20)**: 단일 에이전트는 L3이 최대다. 다중 에이전트 consensus가 있어야 L4/L5가 가능하다.

**Sybil 차단(D26)**: Graph 패턴에서 consensus를 구성하는 reviewer들은 `(name, stance)` 쌍이 구별되어야 한다. 같은 이름에 같은 stance를 가진 두 reviewer는 "동일 에이전트를 복제한 것"으로 간주해 consensus로 인정하지 않는다.

## 엔티티 (DB)

독립 테이블이 없다. decisions 테이블의 `kind = 'consensus'`인 행으로 표현된다.

## API 표면

Consensus 결정 생성 시 M3 단계 검증이 실행된다. proposal_id 미제공 또는 critique 부재 시 에러 반환. 결정-resolver specialist가 주로 이 흐름을 자동화한다.

## 불변식

- **critique 선행 필수(D20)**: 같은 proposal_id를 가진 critique가 최소 1개 없으면 consensus 기록 불가.
- **sybil 차단(D26)**: Graph 패턴 내 reviewer는 (name, stance) 튜플 기준으로 2개 이상의 고유 쌍이 있어야 한다.
- **dissensus는 항상 에스컬레이션**: dissensus 결정은 autonomy mode에 관계없이 인간 게이트로 에스컬레이션된다.

## 구현 위치 (provenance)

M3 단계 전이 검증(`Decision::validate_stage_transition`)은 `sdi-plugin/crates/core/src/decision.rs`에 있다. D26 sybil 차단(`validate_pattern_shape` — graph case)은 `sdi-plugin/crates/core/src/pattern.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: L4 timed-gate(timeout_ms)가 실제로 daemon에서 어떻게 구현되는지 — AutonomyPolicy의 timeout_ms 필드는 존재하나 daemon의 auto-apply 타이머 구현 확인 필요.
