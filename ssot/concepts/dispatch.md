---
id: concept.dispatch
kind: Concept
title: Dispatch(에이전트 디스패치)
definition: "오케스트레이터(메인 Claude Code 세션, Layer 1)가 전문가 서브에이전트(Layer 2 specialist)를 특정 작업에 배정하는 행위. D21(의무적 위임 게이트)에 의해 오케스트레이터는 직접 코드를 편집하지 못하며, 반드시 서브에이전트에게 위임해야 한다."
relatesTo:
  - { to: concept.agent-spec, type: relates-to, note: "디스패치 대상은 AgentSpec에 등록된 전문가 에이전트들이다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "어떤 전문가를 어떤 순서로 디스패치할지가 CollaborationPattern에 선언된다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "D21 의무적 위임 게이트는 autonomy mode와 함께 PreToolUse 훅에서 강제된다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/plugin
  - sdi-plugin/crates/core/src/agent_spec.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Dispatch(디스패치)는 오케스트레이터가 특정 작업을 전문가 서브에이전트에게 위임하는 행위다. SDI의 다중 에이전트 아키텍처에서 오케스트레이터(메인 Claude Code 세션)는 "누가 무엇을 할지"를 결정하는 Layer 1이고, 실제 코드 편집·파일 쓰기·테스트 실행 등 구현 작업은 Layer 2 전문가들이 수행한다.

D21(의무적 위임 게이트)은 이 분리를 강제한다. PreToolUse 훅이 `Edit`/`Write`/`NotebookEdit`/mutating `Bash` 도구 호출을 감지할 때, `hookInput.agent_id`가 서브에이전트(Agent-spawned specialist)를 나타내지 않으면 차단한다. 즉 오케스트레이터가 직접 코드를 편집하려 하면 훅이 막는다.

전문가 서브에이전트는 두 집합으로 구성된다.

**8개 기본 전문가(stock agents)**:
- gwt-converter: 자유 서술 행동을 GWT 시나리오로 변환
- scenario-decomposer: 확정 시나리오를 태스크로 분해
- impl-coder: 코드 구현
- test-runner: 테스트 실행 및 Evidence 기록
- regression-runner: 이전 라운드 시나리오 회귀 검증
- disruption-analyst: 변경이 기존 시나리오에 미치는 영향 분석
- decision-resolver: M3 협상 흐름 주도
- schema-architect: 아키텍처/스키마 제안 비평(D17 강제 L4 대상)

**3개 v0.5 메타 전문가(stock meta-agents)**:
- pattern-orchestrator(proposer stance): 적절한 협업 패턴 제안
- pattern-critic(devil_advocate stance): 패턴 제안 비평
- reversal-runner(neutral stance): D28 롤백 실행

## 엔티티 (DB)

Dispatch 자체는 별도 엔티티가 아니다. AgentSpec 테이블에 전문가 등록 정보가 있고, agent_notes(kind=handoff)로 인계가 이루어지며, collaboration_patterns에 어떤 전문가들을 어떤 순서로 쓸지 선언된다.

## API 표면

디스패치는 오케스트레이터가 `Agent` 도구를 사용해 서브에이전트를 스폰할 때 이루어진다. PreToolUse 훅이 D26 패턴 형태 advisory와 D21 위임 게이트를 동시에 검사한다.

## 불변식

- **D21 의무 위임**: 오케스트레이터는 Edit/Write/NotebookEdit/mutating Bash를 직접 실행할 수 없다. 서브에이전트 context에서만 허용.
- **등록된 전문가만(M5)**: STOCK_AGENTS와 STOCK_META_AGENTS에 있는 이름만 유효한 AgentSpec name이다.
- **instance_count 상한(M5)**: 하나의 AgentSpec은 최대 16개의 동시 인스턴스를 가질 수 있다.

## 구현 위치 (provenance)

AgentSpec 도메인 규칙(name 검증, instance_count 상한)은 `sdi-plugin/crates/core/src/agent_spec.rs`에 있다. D21 PreToolUse 훅은 `sdi-plugin/plugin/` 하위에 있다.

## 미확정 (OPEN)

- [ ] OPEN: D21 훅이 `hookInput.agent_id`를 어떻게 파싱해 서브에이전트 여부를 판별하는지 구체 구현 확인 필요.
