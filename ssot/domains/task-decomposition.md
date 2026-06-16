---
id: domain.task-decomposition
kind: Domain
title: 태스크 분해 (Task Decomposition)
purpose: "LLM 에이전트가 confirmed 시나리오와 요구사항을 읽어 런타임 태스크를 자율 생성하고, 태스크 트리(부모-자식)와 패턴 프로비넌스를 관리한다. 빌더는 태스크를 직접 만들지 않는다."
definition: "Task 엔티티의 런타임 생성·분해·상태 전이를 담당하는 경계. 태스크는 인간이 직접 작성하는 백로그 항목이 아니라 LLM이 시나리오에서 도출하는 런타임 산출물이다(D3). 태스크는 반드시 하나 이상의 시나리오에 연결되어야 하며, 완료 시 구조화된 증거가 필수다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: 태스크는 반드시 하나 이상의 시나리오에서 도출된다. 시나리오가 태스크의 존재 이유이자 완료 기준이다.
  - to: domain.requirements
    type: relates-to
    note: scenario-decomposer가 시나리오와 함께 요구사항을 읽어 태스크 범위를 결정한다.
  - to: domain.round-execution
    type: relates-to
    note: 태스크는 라운드에 귀속된다. 라운드가 활성화되어야 태스크가 생성·실행될 수 있다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: 모든 태스크 생성은 produced_via_pattern_id를 가져야 한다. 패턴 없이 생성하면 자동으로 direct(안티패턴 마커)가 부여된다.
  - to: concept.task
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.task-evidence
    type: relates-to
    note: 태스크 완료의 필수 조건이다.
  - to: concept.evidence-tsv
    type: relates-to
    note: 증거는 구조화된 형식(시나리오별 판정 + 파일:라인 또는 CI URL)으로 제출된다.
governedBy: []
realizedBy: []
impacts:
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/core/src/task.rs"
  - "sdi-plugin/crates/daemon/src/router/task.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI의 핵심 전제 하나를 구현한다 — "태스크는 사람이 만들지 않는다, LLM이 만든다(D3)." Jira나 Linear에서 개발자가 스토리·태스크를 직접 작성하는 것과 달리, SDI에서 태스크는 LLM 에이전트가 확인된 시나리오와 요구사항을 읽고 자율적으로 도출하는 런타임 산출물이다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **태스크 자율 생성**: scenario-decomposer 서브에이전트가 confirmed 시나리오를 읽어 1~3개 태스크를 제안한다. 각 태스크는 하나의 라운드에 귀속되고 하나 이상의 시나리오에 연결된다. 요구사항도 함께 참조할 수 있다.

- **태스크 트리**: 하나의 태스크가 너무 크면 `sdi task decompose`로 하위 태스크로 나눈다. 부모-자식 트리 구조로 복잡한 작업을 계층화한다.

- **상태 전이**: `todo → in_progress → done | cancelled | blocked`. 외부 의존이 있으면 `blocked`. 완료(`done`)로 전환하려면 증거가 필수다 — 근거 없는 완료는 시스템이 거부한다.

- **증거 구조**: 태스크 완료 시 연결된 각 시나리오에 대해 판정(`passing | failing | impacted | retired`)과 증거 참조(파일:라인 번호 또는 로그/CI URL)를 구조화해 제출한다. 단순 텍스트 설명은 증거로 인정되지 않는다.

- **패턴 프로비넌스(D23)**: 태스크 생성 시 어떤 CollaborationPattern에서 나왔는지 기록한다. 패턴 ID를 지정하지 않으면 데몬이 자동으로 `direct`(안티패턴 마커, L3 캡)를 부여한다.

포함되지 않는 것: 시나리오 자체의 관리(domain.scenario-management), 라운드 생성·진행(domain.round-execution), 패턴의 선택·구체화(domain.collaboration-patterns).

## 기능

- 시나리오와 요구사항을 읽어 태스크를 생성한다.
- 태스크를 하위 태스크로 분해한다.
- 태스크 상태를 전이한다.
- 완료 시 시나리오별 구조화 증거를 제출하고 검증한다.
- 태스크에 패턴 프로비넌스를 기록한다.

## 시스템 흐름

라운드가 활성화되면 scenario-decomposer가 확인된 시나리오 목록을 조회하고 각각에 대한 태스크를 제안한다. pattern-orchestrator로부터 받은 active 패턴 ID를 `--produced-via-pattern`으로 지정해 태스크를 생성한다. impl-coder가 태스크를 `in_progress`로 전환하고 구현을 완료하면, test-runner가 각 연결 시나리오에 대한 판정과 증거를 수집해 `sdi task complete`로 제출한다. 증거가 완전하면 태스크가 `done`으로 전환되고, 데몬이 상위 라운드 진행 상황을 갱신한다.

## 다른 도메인과의 관계

태스크는 시나리오와 라운드 사이의 실행 다리다. 시나리오가 "무엇이 되어야 하는가"를 정의하면, 태스크가 "그것을 이루기 위해 무엇을 할 것인가"를 정의하고, 증거가 "실제로 이루어졌는가"를 증명한다. 이 세 층이 맞물려야 SDI의 자동 회귀가 의미를 가진다.

## 미확정 (OPEN)
- [ ] OPEN: 태스크 완료 시 증거가 상위 라운드의 시나리오 판정 결과로 자동 전파되는 정확한 데몬 내부 흐름 확인 필요.
