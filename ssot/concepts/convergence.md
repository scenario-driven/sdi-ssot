---
id: concept.convergence
kind: Concept
title: Convergence(수렴)
definition: "협업 패턴 또는 라운드가 목표에 도달해 완결되는 상태. 패턴의 경우 lifecycle=converged, 라운드의 경우 status=completed가 수렴을 표현한다. Dissensus(불합의)는 수렴 실패로 인간 에스컬레이션을 일으킨다."
relatesTo:
  - { to: concept.collaboration-pattern, type: relates-to, note: "패턴 lifecycle=converged는 협업이 합의에 도달했음을 의미한다." }
  - { to: concept.round, type: relates-to, note: "라운드 status=completed는 검증이 완결됨을 의미한다." }
  - { to: concept.consensus, type: relates-to, note: "Consensus Decision이 나오면 패턴이 converged로 전환될 수 있다." }
  - { to: domain.decision-negotiation, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/pattern.rs
  - sdi-plugin/crates/core/src/round.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Convergence(수렴)는 SDI에서 두 가지 맥락에서 사용된다.

**협업 패턴의 수렴**: 패턴(CollaborationPattern)이 `pending → active`를 거쳐 `converged` 상태에 도달하면 이 패턴 아래서 진행된 협업이 성공적으로 완결되었음을 의미한다. Consensus Decision이 accepted되면 패턴이 converged로 전환된다. 반면 합의가 이루어지지 않으면 `dissensus`로 전환되고 인간 게이트로 에스컬레이션된다.

**라운드의 수렴**: 라운드가 `active → completed`로 전환되면 해당 라운드의 시나리오 검증이 완결된 것이다.

수렴의 반대는 두 가지다.

- **Dissensus**: 에이전트들이 합의에 도달하지 못함. 항상 인간 게이트로 에스컬레이션.
- **Aborted**: 패턴이 중간에 중단됨(이유는 다양).

SDI에서 수렴이 중요한 이유는 "에이전트가 자율적으로 결정을 내리는 것"이 가능한 조건이기 때문이다. Consensus(수렴)가 있어야 L4/L5 autonomy mode에서 자동 적용이 가능하다.

## 엔티티 (DB)

수렴은 독립 테이블이 없다. `PatternLifecycle::Converged`와 `RoundStatus::Completed` 열거값으로 표현된다.

## API 표면

패턴과 라운드 각각의 완료 API를 통해 수렴 상태로 전환된다.

## 불변식

- **Dissensus는 항상 에스컬레이션**: 패턴 dissensus는 autonomy mode에 무관하게 인간 게이트를 요구한다.
- **수렴 후 비가역**: converged/dissensus/aborted 상태는 terminal이며 되돌릴 수 없다. 새 패턴을 만들어야 한다.

## 구현 위치 (provenance)

`PatternLifecycle::is_terminal()` 메서드는 `sdi-plugin/crates/core/src/pattern.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: 패턴 converged 전환의 트리거가 consensus Decision 생성인지, 별도 API 호출인지 확인 필요.
