---
id: domain.disruption-review
kind: Domain
title: 파괴 검토 (Disruption Review)
purpose: "새 시나리오·요구사항·결정이 기존에 passing이었던 시나리오를 무효화하거나 수정해야 하는 상황을 탐지하고, 기본적으로 사람(빌더) 확인을 거쳐 의도적 변경과 의도치 않은 회귀를 구분한다."
definition: "Disruption 정책(D9)의 탐지·분류·처리를 담당하는 경계. 기본 정책은 needs-review — 잠재적 파괴가 감지되면 항상 빌더 확인을 요청한다. '의도된 행동 변경(impacted)'과 '의도치 않은 회귀(failing)'를 명확히 구분한다. 의도된 변경은 Decision 엔티티의 뒷받침이 필요하다."
servesPersona:
  - persona.solo-builder
  - persona.subagent
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: 파괴 탐지의 대상이 기존 시나리오의 판정 상태다. impacted 판정이 이 도메인에서 나온다.
  - to: domain.round-execution
    type: relates-to
    note: 라운드 시작 시 in-flight 태스크가 있으면 파괴 분석이 실행된다(D10).
  - to: domain.decision-negotiation
    type: relates-to
    note: 의도적 행동 변경(impacted)은 반드시 Decision 엔티티의 뒷받침이 있어야 한다. Decision 없이는 impacted가 아니라 failing으로 처리된다.
  - to: concept.disruption-review
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.scenario-claim
    type: relates-to
    note: 시나리오 자원 클레임 충돌이 disruption의 한 형태다.
governedBy: []
realizedBy: []
impacts:
  - domain.scenario-management
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/core/src/disruption.rs"
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
  - "sdi-plugin/plugin/agents/disruption-analyst.md"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 "새 코드가 이전에 만든 약속을 깨뜨렸는가"를 추적한다. SDI에서 시나리오는 동작 약속이다. 새 기능을 추가하다가 기존 시나리오가 깨지면, 그게 의도된 변경인지(정책이 바뀌었다) 실수인지(버그다)를 구분해야 한다. 이 도메인이 그 판단의 근거를 만든다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **needs-review 기본 정책(D9)**: 파괴 가능성이 탐지되면 자동으로 처리하지 않고 반드시 빌더 확인을 요청한다. `auto` 옵션을 선택해도 실제 적용 전 빌더 확인은 필수다.

- **두 가지 판정의 구분**:
  - **impacted**: 의도적으로 기존 시나리오의 동작을 바꾼 것. 반드시 Decision 엔티티가 뒷받침해야 한다. Decision 없이 impacted를 주장하면 failing으로 처리된다.
  - **failing**: 의도치 않은 회귀. 이전에 passing이었는데 현재 코드에서 깨진 것.

- **탐지 방법**: disruption-analyst 서브에이전트가 변경된 파일을 분석해 다른 시나리오가 참조하는 코드 영역과 겹치는지 확인한다. 겹치는 시나리오는 파괴 후보가 된다.

- **라운드 시작 시 in-flight 점검**: 이미 진행 중인 태스크가 있는 상태에서 새 라운드를 시작하면, 진행 중 작업이 기존 시나리오에 영향을 줄 수 있는지 검토가 이루어진다(D10).

포함되지 않는 것: 회귀 시나리오의 재실행 자체(domain.round-execution의 regression-runner), 결정 내용 작성(domain.decision-negotiation), 시나리오 상태 기록(domain.scenario-management).

## 기능

- 변경 범위와 기존 시나리오의 자원 클레임을 대조해 파괴 후보를 탐지한다.
- 파괴 후보마다 의도적 변경인지 회귀인지 판단 게이트를 연다.
- 의도적 변경이면 Decision을 통한 뒷받침을 요청한다.
- impacted 판정을 Decision ID와 연결해 기록한다.
- AgentNote에 파괴 경고를 기록해 다른 에이전트들이 참조할 수 있게 한다.

## 시스템 흐름

test-runner나 impl-coder가 기존 시나리오가 깨진 것을 감지하면 disruption-analyst 서브에이전트를 호출한다. analyst가 변경 파일과 시나리오 참조 영역을 대조해 파괴 후보 목록을 만든다. 각 후보에 대해 "의도된 변경이면 Decision을 만들어라"고 decision-resolver에 핸드오프하거나, "의도치 않은 회귀면 failing으로 처리하라"고 test-runner에게 돌려보낸다. 빌더가 최종 판단을 확인한다(needs-review 기본).

## 다른 도메인과의 관계

파괴 검토는 새 개발과 기존 품질 보증 사이의 완충지대다. 라운드 실행이 새로운 시나리오를 처리하는 전진 방향이라면, 파괴 검토는 그 전진이 기존 시나리오를 무너뜨리지 않도록 확인하는 후방 점검이다.

## 미확정 (OPEN)
- [ ] OPEN: disruption.rs 코어 모듈의 현재 구현 범위(탐지 알고리즘이 파일 참조 분석인지 시나리오 메타데이터 대조인지) 확인 필요.
