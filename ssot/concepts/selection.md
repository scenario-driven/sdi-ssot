---
id: concept.selection
kind: Concept
title: Selection(에이전트 선택)
definition: "오케스트레이터가 작업에 적합한 전문가 에이전트를 선택하는 판단 과정. pattern-orchestrator specialist가 CollaborationPattern을 제안하고, AgentSpec 정보를 바탕으로 어떤 에이전트를 어떤 순서·역할로 투입할지 결정한다."
relatesTo:
  - { to: concept.agent-spec, type: relates-to, note: "선택 대상은 AgentSpec에 등록된 전문가들이다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "pattern-orchestrator가 CollaborationPattern으로 선택 결과를 선언한다." }
  - { to: concept.dispatch, type: relates-to, note: "Selection 후 선택된 에이전트들을 Dispatch한다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/agent_spec.rs
  - sdi-plugin/crates/core/src/pattern.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Selection(선택)은 주어진 작업에 어떤 전문가 에이전트들을 어떤 방식으로 투입할지 결정하는 과정이다. SDI에서 선택은 단순히 "이 에이전트를 쓴다"가 아니라 **CollaborationPattern을 통해 구조화된 협업 방식을 선언**하는 것을 포함한다.

pattern-orchestrator specialist가 선택의 주체다. 작업의 성격(복잡도, 리스크, 도메인)을 분석해 적합한 패턴 종류(workflow/graph/swarm/agents-as-tools)와 구체적인 에이전트 구성을 제안한다. pattern-critic specialist가 이 제안을 비평하고 D26 형태 게이트가 최소 요건을 검사한다.

선택 결과는 CollaborationPattern 행으로 저장된다. 이후 디스패치 시 오케스트레이터는 이 패턴을 따라 에이전트들을 순서대로 또는 병렬로 스폰한다.

## 엔티티 (DB)

독립 엔티티 없음. 선택 결과는 collaboration_patterns 테이블에 저장된다.

## API 표면

pattern-orchestrator가 /patterns API를 통해 패턴을 생성한다.

## 불변식

- **등록된 에이전트만 선택 가능(M5)**: AgentSpec에 없는 이름은 패턴에 포함할 수 없다.

## 구현 위치 (provenance)

`STOCK_AGENTS`, `STOCK_META_AGENTS`는 `sdi-plugin/crates/core/src/agent_spec.rs`에 있다. 패턴 구조(`PatternStep`, `Reviewer`, `PeerLink`)는 `sdi-plugin/crates/core/src/pattern.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: pattern-orchestrator가 어떤 기준으로 특정 패턴 종류를 선택하는지 — PRD 또는 skill 파일에서 선택 기준 확인 필요.
