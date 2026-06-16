---
id: invariant.circuit-breaker-always-on
kind: Invariant
title: 사용자는 언제든 단일 액션으로 모든 autonomy mode를 즉시 L3로 강등할 수 있다
definition: "circuit breaker는 항상 활성 상태여야 한다. 사용자가 UI 단일 액션(대시보드 버튼, 전역 단축키, 또는 CLI)을 실행하면 모든 AutonomyPolicy 행의 mode가 즉시 L3(always ask)로 강등된다. 이 강등은 즉시 발효되며, in-flight 결정은 다음 게이트에서 사람 확인을 거친다. 어떤 설정으로도 circuit breaker를 비활성화할 수 없다."
governs:
  - domain.autonomy
  - domain.decision-negotiation
  - domain.agent-coordination
  - concept.autonomy-policy
  - concept.circuit-breaker
  - concept.decision
  - concept.collaboration-pattern
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/autonomy_policy.rs"
  - "sdi-plugin/docs/ARCHITECTURE.md"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 자율 실행(L4/L5)은 사용자가 자리를 비워도 작업이 진행된다는 전제 위에 있다. 이 전제가 성립하려면 사용자가 "도구가 너무 자율적"이라고 느끼는 순간 즉시, 완전하게, 그리고 단순하게 통제권을 되찾을 수 있어야 한다. circuit breaker는 그 안전 장치다(D18).

circuit breaker가 발동되면 다음이 일어난다:
- 모든 `AutonomyPolicy` 행의 `mode`가 `L3`로 일괄 업데이트된다
- `set_by='circuit-breaker'`로 표기되어 circuit breaker가 발동한 사실이 기록된다
- 이미 진행 중인 consensus는 취소되지 않고 다음 사용자 게이트 도달 시 L3 확인을 받는다
- 패턴별 policy, plan별 policy, decision_kind별 policy 전부가 L3로 강등된다

트리거 조건은 사용자의 명시 액션 외에 자동 트리거도 있다: dissensus 누적, M5 spawn 루프, AgentNote 쓰기 속도 급증. 이 자동 트리거 역시 즉시 L3 강등을 일으킨다.

circuit breaker를 끄는 설정이나 env var는 존재하지 않는다. 자율성 모드를 복원하려면 사용자가 AutonomyPolicy를 명시적으로 L4/L5로 재설정해야 한다.

## 깨지면 무슨 일이 일어나나

circuit breaker가 없거나 비활성화될 수 있다면, 자율 실행이 사용자의 의도를 벗어난 방향으로 진행될 때 통제권을 되찾는 공식 경로가 없다. L5 모드에서 blast_radius_score가 높은 architecture 결정이 자동 적용되고 있는데 사용자가 이를 멈추기 어려워진다. 특히 여러 에이전트가 병렬로 consensus를 형성하며 연쇄적으로 L5 결정을 실행하는 상황에서 circuit breaker 없이는 전체 실행을 멈출 단일 액션이 없다. 이는 D16의 "사용자가 중간 개입 구멍을 언제든 토글"이라는 약속을 깨는 것이다.

## 코드에서 어떻게 강제되나

데몬의 circuit breaker 엔드포인트(`POST /autonomy_policies/circuit_breaker`)는 단일 트랜잭션으로 모든 `autonomy_policy` 행을 `mode='L3', set_by='circuit-breaker'`로 업데이트한다. `sdid`가 이 엔드포인트를 항상 노출하며 비활성화 플래그가 없다. SSE 이벤트 `circuit_breaker_tripped`가 모든 연결된 세션(대시보드 SPA, sdi-desktop)에 즉시 전파된다. 대시보드와 desktop은 이 이벤트를 수신하면 모든 L4/L5 UI 요소를 즉시 L3 표시로 갱신한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D18 Circuit breaker 항시 활성 결정 엔트리) 연결 필요
