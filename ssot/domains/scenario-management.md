---
id: domain.scenario-management
kind: Domain
title: 시나리오 관리 (Scenario Management)
purpose: "자연어 Given/When/Then 시나리오를 SDI의 1등 시민으로 생성·확인·태그·의존관계 관리하고, 라운드별 검증 결과(passing/failing/impacted/retired)를 누적한다."
definition: "플랜에 귀속된 Scenario 엔티티의 CRUD, GWT 형식 강제, 상태 전이(draft→confirmed→라운드별 판정), 의존관계 DAG 관리, 자원 클레임(D29)을 담당하는 경계. 자유 형식 텍스트 시나리오는 이 도메인이 거부한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.requirements
    type: relates-to
    note: 요구사항(Requirement)이 시나리오의 컨텍스트를 제공한다. 태스크는 시나리오+요구사항을 함께 보고 분해된다.
  - to: domain.round-execution
    type: relates-to
    note: 라운드가 시나리오를 집행 단위로 사용한다. R1은 confirmed 시나리오를 구현하고, R2+는 passing 시나리오를 회귀 재실행한다.
  - to: domain.disruption-review
    type: relates-to
    note: 기존 시나리오에 영향을 주는 변경이 생기면 이 도메인의 시나리오 상태가 impacted 후보로 표시되어 검토된다.
  - to: concept.scenario
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.given-when-then
    type: relates-to
    note: 모든 시나리오가 준수해야 하는 형식 제약이다.
  - to: concept.scenario-id
    type: relates-to
    note: 시나리오 식별자 체계(SC-N 형식).
  - to: concept.regression
    type: relates-to
    note: 이전 라운드 passing 시나리오의 자동 회귀 재실행이 이 도메인의 데이터를 기반으로 한다.
governedBy: []
realizedBy: []
impacts:
  - domain.round-execution
  - domain.disruption-review
implementedIn:
  - "sdi-plugin/crates/core/src/scenario.rs"
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI의 핵심 전제를 구현한다 — "자연어 시나리오가 1등 시민이다." TDD에서 테스트 코드가, BDD에서 Gherkin 스텝 정의가 하던 역할을, SDI에서는 자연어 GWT 시나리오가 한다. 이 도메인이 그 시나리오의 생애 전체를 관리한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **시나리오 생성과 GWT 강제**: 빌더 또는 gwt-converter 서브에이전트가 Given/When/Then을 제시하면 이 도메인이 세 필드가 모두 비어 있지 않은지 검증한다. 자유 형식 옵션은 없다(D5). 자연어를 GWT로 변환하는 것은 LLM 보조이지 이 도메인의 책임이 아니다.

- **상태 전이**: 시나리오는 `draft → confirmed`를 거쳐 라운드별 판정(`passing | failing | impacted | retired`)을 누적한다. `confirmed`가 되어야 라운드에서 집행된다. 같은 시나리오가 R1에서 passing이고 R2에서 failing일 수 있으며, 이 이력이 회귀 감지의 근거다.

- **의존관계 DAG**: 시나리오끼리 `depends_on` 관계로 선행 조건을 표현할 수 있다. 순환 참조는 데몬이 위상 정렬로 차단한다.

- **자원 클레임(D29)**: 각 시나리오는 자신이 작업할 파일·디렉터리 경로 패턴을 `claimed_resources`로 선언한다. 다른 세션의 시나리오와 클레임이 겹치면 데몬이 충돌을 차단하고 빌더에게 판단을 요청한다.

포함되지 않는 것: 라운드 생성·진행(domain.round-execution), 태스크 분해(domain.task-decomposition), 요구사항 스냅샷(domain.requirements).

## 기능

- 시나리오를 만들고 GWT 형식을 검증한다.
- 시나리오를 draft에서 confirmed로 전환한다(빌더의 명시적 확인).
- 라운드별 판정 결과(verdict)와 증거 참조를 기록한다.
- 시나리오 간 의존관계를 관리하고 DAG 무결성을 보장한다.
- 자원 클레임 상태(`none → requested → active → released`)를 관리하고 충돌을 탐지한다.
- 시나리오에 태그를 붙여 그룹핑한다(D4에서 Unit이 제거되고 태그로 격하된 후신).
- 각 시나리오에 구현 책임 에이전트(`produced_by`)와 검증 책임 에이전트(`verified_by`)를 기록한다.

## 시스템 흐름

빌더나 gwt-converter가 자연어 시나리오를 제시하면, 데몬이 GWT 세 필드의 유효성을 검사한다. 검사를 통과하면 `draft` 상태로 저장된다. 빌더가 의도를 올바르게 표현했다고 판단하면 `confirmed`로 전환한다. 이 confirmed 시나리오들이 충분히 모이고 플랜이 승인되면 라운드가 시작될 수 있다. 라운드 안에서 test-runner 서브에이전트가 각 시나리오에 판정과 증거를 기록하면, 이 도메인의 `per_round_results`가 갱신된다.

## 다른 도메인과의 관계

시나리오 관리는 SDI 전체의 중심이다. 플랜 승인은 시나리오 수에 달려 있고(domain.planning), 라운드는 시나리오를 집행 단위로 삼으며(domain.round-execution), 파괴 감지는 시나리오 상태 변화를 추적한다(domain.disruption-review). 자원 클레임은 다중 세션 협업 안전을 보장하는 공유 자원으로 project-management 도메인과 연결된다.

## 미확정 (OPEN)
- [ ] OPEN: `produced_via_pattern_id` 필드가 시나리오에도 적용되며(D23), 현재 코드에서 시나리오 생성 시 pattern ID 바인딩 검증이 실제로 강제되는지 확인 필요.
