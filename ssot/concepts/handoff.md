---
id: concept.handoff
kind: Concept
title: Handoff(인계)
definition: "M2 통신 레이어 — 에이전트가 다른 에이전트에게 작업을 명시적으로 인계하는 AgentNote의 특별 종류. kind=handoff인 AgentNote를 통해 표현되며, to_agent 필드가 필수다. 인계 메시지를 통해 오케스트레이터(Layer 1)가 다음 실행자를 결정한다."
relatesTo:
  - { to: concept.agent-note, type: backed-by, note: "Handoff는 kind=handoff인 AgentNote로 표현된다. 독립 엔티티가 아니다." }
  - { to: concept.agent-spec, type: relates-to, note: "to_agent는 AgentSpec의 name과 대응한다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/agent_note.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Handoff(인계)는 한 에이전트가 다른 에이전트에게 "이제 네 차례다"라고 명시적으로 알리는 통신 패턴이다. SDI의 다중 에이전트 흐름에서 에이전트들은 오케스트레이터(메인 Claude Code 세션, Layer 1)를 통해 간접적으로 조율될 수 있지만, Handoff를 사용하면 에이전트 A가 에이전트 B에게 직접 인계 신호를 보낼 수 있다.

Handoff는 `kind=handoff`인 AgentNote로 표현된다. `to_agent` 필드에 인계 대상 에이전트 이름을 명시해야 한다. 이 필드가 비어 있으면 도메인 레벨에서 거부된다(M2 규칙).

Handoff 메시지의 body에는 인계 맥락(현재까지 한 작업, 남은 작업, 중요 발견 사항 등)을 담는다. payload JSON에는 구조화된 추가 정보를 포함할 수 있다.

## 엔티티 (DB)

Handoff는 독립 테이블이 없다. agent_notes 테이블에서 `scope_kind`와 `kind = 'handoff'`인 행으로 조회한다.

## API 표면

Handoff 노트는 AgentNote 생성 API에 `kind=handoff`와 `to_agent` 값을 함께 전달해 생성한다.

## 불변식

- **to_agent 필수(M2)**: kind=handoff인 AgentNote는 to_agent가 비어 있으면 안 된다.

## 구현 위치 (provenance)

`AgentNote::validate_handoff` 검증 로직은 `sdi-plugin/crates/core/src/agent_note.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: 오케스트레이터가 handoff 노트를 어떻게 감지해 다음 에이전트를 활성화하는지 — 훅 또는 SSE 이벤트 기반인지 확인 필요.
