---
id: domain.collaboration-patterns
kind: Domain
title: 협업 패턴 (Collaboration Patterns)
purpose: "모든 work entity(플랜·요구사항·시나리오·태스크·결정·라운드)가 어떤 다중 에이전트 협업 형태로 만들어졌는지를 CollaborationPattern 엔티티에 영구 기록하고, 패턴 종류별 무결성 규칙을 강제해 가짜 협업을 차단한다."
definition: "CollaborationPattern 엔티티(D22, SDI 7번째 1등 시민)의 생성·전이·shape 검증을 담당하는 경계. 종류는 workflow·graph·swarm·agents-as-tools·direct 중 하나이며, 각 종류별 shape 기준이 있다. 모든 신규 work entity는 produced_via_pattern_id를 가져야 한다(D23). 패턴 없이 생성된 entity에는 자동으로 direct(안티패턴 마커, L3 캡)가 부여된다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.decision-negotiation
    type: relates-to
    note: Graph 패턴이 결정 협상에 적용되어 distinct (name, stance) ≥ 2 조건으로 sybil을 차단한다.
  - to: domain.autonomy
    type: relates-to
    note: 패턴 종류별 기본 자율 모드가 있다(workflow=L5, swarm=L4, direct=L3 강제). 플랜 모드와 패턴 모드 중 더 엄격한 쪽이 적용된다.
  - to: domain.task-decomposition
    type: relates-to
    note: 태스크 생성 시 produced_via_pattern_id를 지정해야 한다. 지정 없으면 direct 자동 부여.
  - to: concept.collaboration-pattern
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.pattern-lifecycle
    type: relates-to
    note: pending → active → converged | dissensus | aborted 전이 규칙이다.
  - to: concept.convergence
    type: relates-to
    note: 패턴이 converged 상태에 이르면 해당 패턴이 담당한 work entity들이 완성됐다는 신호다.
governedBy: []
realizedBy: []
impacts:
  - domain.autonomy
  - domain.decision-negotiation
  - domain.task-decomposition
implementedIn:
  - "sdi-plugin/crates/core/src/pattern.rs"
  - "sdi-plugin/crates/core/src/collab.rs"
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
  - "sdi-plugin/plugin/agents/pattern-orchestrator.md"
  - "sdi-plugin/plugin/agents/pattern-critic.md"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI가 "단일 에이전트 1인극은 안티패턴(D13)"이라는 원칙을 데이터 수준에서 강제한다. 코드 주석이나 문서에서 "다중 에이전트를 써야 한다"고 권고하는 것이 아니라, 패턴을 DB row로 만들어 모든 work entity가 자신이 어떤 협업 형태로 만들어졌는지 기록하게 한다. 이 기록이 없으면 패턴 integrity gate가 진행을 차단한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **네 가지 유효 패턴(D15)**: 
  - **Workflow**: 고정 순서 단계(step)들을 순차 실행. 단계 ≥ 2 필수. 선행 단계 증거 없이 후행 단계 실행 차단.
  - **Graph**: DAG 의존 관계를 가진 리뷰 구조. 합의 형성을 위해 distinct (에이전트명, stance) 쌍 ≥ 2 필수. 같은 종류의 에이전트 두 인스턴스는 판단 다양성을 보장하지 않으므로 차단(D26 sybil 차단).
  - **Swarm**: 병렬 실행 전문가들. fan_out ≥ 2 필수. 자기 spawn 루프와 depth cap 초과 차단.
  - **Agents-as-Tools**: 하나의 에이전트가 다른 에이전트를 도구로 호출. 피어 등록 ≥ 1 필수.

- **안티패턴 마커(direct)**: 위 네 패턴에 해당하지 않는 1인극. 자동으로 L3 캡(항상 사람 확인)이 걸리고 대시보드에 빨간 배지가 표시된다. 의도적으로 선택해야 하며, 가짜 패턴으로 우회하면 shape gate가 차단한다.

- **생명주기**: `pending → active → converged | dissensus | aborted`. `pending → active` 전이 시 shape 검증을 강제 통과해야 한다. 1-step workflow나 1-agent swarm 같은 가짜 패턴은 active 전환이 거부된다.

- **패턴 재귀(D24)**: 패턴이 하위 패턴을 spawn할 수 있다(`parent_pattern_id` 자기참조). 기본 depth cap 3. DAG만 허용하고 순환 차단.

포함되지 않는 것: 자율성 정책 설정(domain.autonomy), 실제 결정 내용(domain.decision-negotiation), 태스크 내용(domain.task-decomposition).

## 기능

- CollaborationPattern을 만들고 종류와 shape manifest를 등록한다.
- `pending → active` 전이 시 shape 기준 검증을 수행한다.
- 패턴이 적용되는 work entity에 produced_via_pattern_id를 연결한다.
- 없는 패턴으로 entity를 생성하려 할 때 자동으로 direct 패턴을 부여한다.
- 패턴 재귀 depth를 추적하고 cap 초과를 차단한다.
- 패턴 종류별 자율 모드를 조회한다.

## 시스템 흐름

pattern-orchestrator 서브에이전트가 작업 형태를 분석해 적합한 패턴 종류를 제안한다. pattern-critic이 shape 무결성을 독립 감사한다. 비평을 받은 orchestrator가 `pending → active` 전이를 요청하면, 데몬이 D26 shape 기준을 재검증하고 통과하면 active로 전환한다. 이후 work entity 생성 시 이 active 패턴 ID를 `--produced-via-pattern`으로 지정한다.

## 다른 도메인과의 관계

협업 패턴은 SDI의 모든 work entity를 가로지르는 메타 도메인이다. 패턴이 없으면 L3 캡, 패턴이 있으면 패턴 종류에 따른 자율 모드가 적용된다. 자율성 도메인이 "어떤 수준으로 자동화할 것인가"를 정하면, 이 도메인이 "그 자동화가 진짜 협업에 근거하는가"를 검증한다.

## 미확정 (OPEN)
- [ ] OPEN: pattern-critic의 감사가 현재 claude agent 정의 파일로만 존재하는지 daemon에도 하드코딩된 shape validation이 병행하는지 코드 확인 필요.
