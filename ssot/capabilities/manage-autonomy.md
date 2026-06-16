---
id: capability.manage-autonomy
kind: Capability
title: 자율성 정책 설정 및 회로 차단기
purpose: "에이전트가 사람 확인 없이 결정을 적용할 수 있는 범위(L3/L4/L5)를 프로젝트·플랜·결정 종류·패턴 단위로 세밀하게 제어하고, 비상시 전체를 L3으로 즉시 강하한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/autonomy.md
  - sdi-plugin/plugin/commands/autonomy.md
relatesTo:
  - { to: concept.autonomy-policy, type: mutates, note: "L3/L4/L5 모드를 scope(global/plan/decision_kind/pattern_kind) 단위로 설정하고 조회한다" }
  - { to: concept.circuit-breaker, type: mutates, note: "circuit-breaker 명령으로 프로젝트 전체 정책을 L3으로 즉시 강하한다(D18)" }
  - { to: concept.decision, type: reads, note: "결정 종류(architecture/schema/naming-canonical)는 D17에 의해 L4 이상 강제" }
  - { to: concept.collaboration-pattern, type: reads, note: "패턴 종류마다 기본 자율성이 다르며 정책이 이를 오버라이드할 수 있다(D25)" }
  - { to: concept.consensus, type: reads, note: "다중 에이전트 합의가 L4/L5 자동 적용 게이트를 여는 조건 중 하나다(D20)" }
impacts:
  - concept.autonomy-policy
  - concept.circuit-breaker
  - concept.decision
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

에이전트가 얼마나 자율적으로 행동할지를 사람이 조율한다. 세 단계가 있다: L3(항상 확인), L4(중요 결정 종류만 확인), L5(합의되면 자동 적용). 탐색적 새 플랜에서는 L5로 빠르게 진행하고, 운영 직전이나 외부 연동이 있는 경우에는 L4로 낮춘다. 아키텍처·스키마·명칭 표준 결정은 D17에 의해 L5로 설정해도 항상 L4 이상을 유지하도록 강제된다.

비상 상황에서는 회로 차단기(circuit breaker)를 발동하면 프로젝트의 모든 자율성 정책이 한 번의 명령으로 L3으로 강하된다. 이미 진행 중인 결정은 다음 게이트 시점에 L3 규칙을 적용받는다. 회복 후 개별 정책을 다시 원하는 수준으로 복원한다.

## 행위

- `sdi autonomy set <PRJ-ID> --scope global --mode L5` 로 프로젝트 전체 기본값을 설정한다.
- `sdi autonomy set <PRJ-ID> --scope plan --mode L4 --plan-id <ID>` 로 특정 플랜에 오버라이드한다.
- `sdi autonomy set <PRJ-ID> --scope decision_kind --mode L4 --decision-kind architecture` 로 결정 종류별로 강제한다.
- `sdi autonomy get <PRJ-ID>` 와 `sdi autonomy list <PRJ-ID>` 로 현재 정책을 조회한다.
- `sdi autonomy circuit-breaker <PRJ-ID> --reason "…"` 으로 비상 L3 강하를 실행한다.
- L5 를 forced-L4 종류에 설정하면 HTTP 403으로 거부된다.

## 시스템 흐름

정책 설정 → 데몬이 scope 계층(plan > decision_kind > global)에서 가장 구체적인 정책을 해석 → 결정 합의 도달 시 자율성 모드 확인 → L5면 자동 적용, L4면 해당 종류 확인, L3면 항상 확인 → circuit-breaker 발동 시 단 트랜잭션으로 전체 정책 L3 강하 → `circuit_breaker.triggered` 이벤트 에미트.

## 어디에 구현되어 있나

`/autonomy` 슬래시 명령(`plugin/commands/autonomy.md`)과 `autonomy` 스킬(`plugin/commands/autonomy.md`)이 설정·조회·circuit-breaker 절차를 정의한다. 데몬 HTTP API가 정책 저장, scope 우선순위 해석, circuit-breaker 트랜잭션을 처리한다.

## 미확정 (OPEN)

- [ ] OPEN: circuit-breaker 발동 후 개별 정책 복원 시 원래 값이 어딘가에 보존되는지, 아니면 사람이 직접 재설정해야 하는지 구현 확인 필요.
