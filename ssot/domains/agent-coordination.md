---
id: domain.agent-coordination
kind: Domain
title: 에이전트 조율 (Agent Coordination)
purpose: "다중 에이전트가 가설·관찰·질문·이견·증거·핸드오프를 AgentNote 블랙보드에 공유하고, AgentSpec으로 전문 역할을 등록해 서로가 누구인지 알고 협업하게 한다."
definition: "AgentNote(M1 블랙보드, M2 핸드오프)와 AgentSpec(M5 자기조직화)를 담당하는 경계. AgentNote는 append-only 저널로 에이전트 간 단기 사고 공유 채널이다. AgentSpec은 런타임에 에이전트가 자신의 이름·stance를 등록해 Graph 패턴의 distinct 조건 검증에 사용된다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.knowledge-rag
    type: relates-to
    note: AgentNote가 단기 에이전트 간 채널이면, Knowledge RAG가 세션을 넘는 장기 지식 채널이다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: AgentSpec의 (name, stance) 등록이 Graph 패턴의 sybil 차단 조건 검증에 사용된다.
  - to: domain.plugin-runtime
    type: relates-to
    note: 오케스트레이터가 MCP 도구를 통해 AgentNote를 추가하고 조회한다.
  - to: concept.agent-note
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.handoff
    type: relates-to
    note: to_agent 필드가 있는 AgentNote는 특정 에이전트로의 핸드오프 신호다.
governedBy: []
realizedBy: []
impacts:
  - domain.collaboration-patterns
implementedIn:
  - "sdi-plugin/crates/core/src/agent_note.rs"
  - "sdi-plugin/crates/core/src/agent_spec.rs"
  - "sdi-plugin/crates/daemon/src/router/agent_note.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 에이전트들이 서로 무엇을 생각하고 무엇을 발견했는지 공유하는 채널을 제공한다. 분리된 서브에이전트들이 각자 독립적으로 실행되지만, 블랙보드(AgentNote)와 자기소개(AgentSpec)를 통해 조율된 협업이 가능해진다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **AgentNote — M1 블랙보드**: 에이전트가 자신의 사고 과정(가설·관찰·질문·이견·증거)을 기록하는 공유 저널. append-only로 삭제가 없다. 종류(kind)는 `hypothesis | observation | question | dissent | evidence | handoff` 중 하나.

- **AgentNote — M2 핸드오프**: `to_agent` 필드가 있는 AgentNote는 특정 에이전트로의 작업 이전 신호다. 예: impl-coder가 구현을 마치고 `to_agent=test-runner`로 핸드오프 노트를 남기면, test-runner가 이를 보고 검증을 시작한다. 수신 확인은 `receipt_acknowledged_at`으로 기록한다.

- **AgentSpec — M5 자기조직화**: 에이전트가 런타임에 자신의 이름과 stance를 등록한다. stance는 `proposer | devil_advocate | schema_guardian | performance_reviewer | security_reviewer | neutral` 중 하나. 이 등록이 Graph 패턴에서 "(이름, stance) 쌍 고유성 ≥ 2" 조건 검증에 쓰인다. `expires_at` 도달 또는 명시 폐기로 `expired` 전환.

포함되지 않는 것: 세션 간 지식 저장(domain.knowledge-rag), 패턴 shape 검증(domain.collaboration-patterns), 결정 ADR(domain.decision-negotiation).

## 기능

- 에이전트가 AgentNote를 추가한다(스코프: plan·round·scenario·task 중 하나에 귀속).
- 핸드오프 노트(to_agent 지정)를 보내고 수신 확인을 기록한다.
- AgentSpec을 등록하고 만료를 관리한다.
- 스코프별 AgentNote 목록을 조회한다.

## 시스템 흐름

서브에이전트가 작업을 마치면 핸드오프 AgentNote를 남긴다. 다음 서브에이전트가 자신을 향한 핸드오프가 있는지 확인하고 작업을 이어받는다. pattern-orchestrator가 패턴을 만들기 전에 참여 에이전트들이 AgentSpec으로 자신의 이름과 stance를 등록한다. Graph 패턴의 합의 단계에서 데몬이 AgentSpec 등록 내역을 조회해 distinct 조건을 검증한다.

## 다른 도메인과의 관계

에이전트 조율은 SDI의 다중 에이전트 통신 기반 구조(M1~M5)의 두 축을 담당한다. 단기 채널(AgentNote)과 자기소개(AgentSpec)를 통해 에이전트들이 고립된 실행자가 아니라 서로를 아는 팀으로 협업한다.

## 미확정 (OPEN)
- [ ] OPEN: AgentSpec의 만료 시간 기본값과 자동 갱신 메커니즘이 코드에서 어떻게 구현되어 있는지 확인 필요.
