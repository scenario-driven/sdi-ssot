---
id: component.agent-pattern-orchestrator
kind: SystemComponent
title: pattern-orchestrator 서브에이전트
purpose: 새로운 작업 엔티티(플랜·요구사항·시나리오·태스크·결정·라운드)가 생성될 때 적합한 CollaborationPattern을 제안하고 등록하는 전문 에이전트다. 모든 작업 엔티티가 실질적인 패턴 근거(produced_via_pattern_id)를 갖추도록 보장해 `direct`(단독 흐름) 안티패턴을 방지한다.
realizedBy:
  - domain.collaboration-patterns
  - domain.agent-coordination
  - domain.planning
implementedIn:
  - sdi-plugin/plugin/agents/pattern-orchestrator.md
dependsOn:
  - component.daemon
  - component.cli
  - component.agent-pattern-critic
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-pattern-critic
    type: leads-to
    note: 패턴 생성(pending) 후 형상 검증을 critic에게 위임한다
  - to: domain.collaboration-patterns
    type: backed-by
  - to: domain.agent-coordination
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

pattern-orchestrator는 입장(stance)이 `proposer`인 에이전트로, SDI 다중 에이전트 협업의 출발점을 구성한다. 새 플랜·시나리오·태스크·결정·라운드가 생성될 때 해당 작업에 어울리는 `CollaborationPattern`이 이미 활성화되어 있는지 확인한다. 없으면 새 패턴을 `pending` 상태로 생성하고, pattern-critic에게 형상 심사를 위임한 뒤, 통과하면 `active`로 전환한다.

패턴 종류 선택 기준은 작업 형상에 따른다. 단계 간 명시적 핸드오프가 있는 순차 흐름(초안→비평→검증)에는 `workflow`, 최소 2명의 다른 에이전트가 독립적으로 검토하는 결정에는 `graph`, N개 전문 에이전트가 병렬로 실행되는 실행 흐름에는 `swarm`, 호출자 에이전트가 특정 피어 에이전트에 단방향 접근해야 할 때는 `agents-as-tools`를 선택한다. `direct`는 진정한 단독 작업을 명시적으로 표시하는 L3 상한 배지이지 기본값이 아니다.

중첩 패턴(parent_pattern_id)도 관리한다. 한 패턴의 단계에서 하위 패턴을 스폰할 수 있으며, 깊이는 AutonomyPolicy의 `pattern_depth_cap`(기본 3)을 초과할 수 없다.

## 경계와 의존

Bash·Read·Grep·Glob 도구로 기존 패턴 상태를 파악하고, Skill·SendMessage·TaskUpdate·TaskList·TaskGet으로 워크플로우를 구동한다. `sdi pattern create` CLI 명령으로 패턴 행을 생성하고, critic 심사 완료 후 `sdi pattern activate` 명령으로 전환한다.

## 통신 패턴

메인 세션 오케스트레이터 또는 훅 어댑터의 D13 분해 권고로 활성화된다. 패턴 활성화 완료 후 결과를 메인 세션에 반환한다. pattern-critic과의 통신은 SendMessage 또는 서브에이전트 스폰으로 이뤄진다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/pattern-orchestrator.md`)로 구성된다. 패턴 종류 선택 기준·워크플로우·critic 위임 절차가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: 패턴 `pending → active` 전환 시 critic 심사가 필수인지, 특정 조건(단순 direct 패턴 등)에서는 건너뛸 수 있는지 확인 필요.
