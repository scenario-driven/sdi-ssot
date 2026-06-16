---
id: concept.agent-note
kind: Concept
title: AgentNote(에이전트 노트)
definition: "M1 블랙보드 — 에이전트 간 비동기 통신을 위한 append-only 저널. 범위(plan/round/scenario/task)에 닻을 내리며 종류(hypothesis/observation/question/dissent/evidence/handoff)로 분류된다. 절대 삭제되지 않으며 retire_at으로만 비활성화된다."
relatesTo:
  - { to: concept.plan, type: relates-to, note: "scope=plan인 노트는 플랜 전체에 적용되는 메모다." }
  - { to: concept.round, type: relates-to, note: "scope=round인 노트는 특정 라운드의 에이전트 통신이다." }
  - { to: concept.scenario, type: relates-to, note: "scope=scenario인 노트는 특정 시나리오에 대한 에이전트 의견이다." }
  - { to: concept.task, type: relates-to, note: "scope=task인 노트는 태스크 실행 중 에이전트 통신이다." }
  - { to: concept.handoff, type: relates-to, note: "kind=handoff인 노트는 to_agent가 필수이며 작업 인계를 의미한다." }
  - { to: concept.agent-spec, type: relates-to, note: "노트의 from_agent와 to_agent는 AgentSpec의 name과 대응한다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/agent_note.rs
  - sdi-plugin/crates/db/src/migrations/006_v04_multi_agent.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

AgentNote(에이전트 노트)는 SDI의 M1 통신 레이어로, 에이전트들이 비동기적으로 서로에게 메시지를 남기는 블랙보드다. 한 번 기록한 노트는 삭제하지 않는다(retain for audit). 더 이상 활성 블랙보드에 노출할 필요가 없어진 노트는 `retired_at` 타임스탬프로 비활성화한다.

노트는 반드시 하나의 닻(anchor)에 연결된다. plan/round/scenario/task 중 정확히 하나만 가리킨다.

**노트 종류(kind)**:

- **hypothesis**: 가설 또는 미검증 아이디어.
- **observation**: 실행 결과나 시스템 상태에 대한 관찰.
- **question**: 답변이 필요한 질문.
- **dissent**: 이의 제기. M3 에스컬레이션을 트리거한다(D20).
- **evidence**: 시나리오 검증 근거.
- **handoff**: 작업 인계. `to_agent` 필드가 필수다(M2 규칙).

dissent 종류는 Decision 흐름의 escalation을 일으킨다. 에이전트가 이의를 제기하면 M3 협상 흐름이 시작되거나 인간 게이트로 에스컬레이션된다.

## 엔티티 (DB)

에이전트 노트 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 프로젝트 참조, 그리고 한 가지 닻(plan/round/scenario/task ID 중 정확히 하나).
- **분류**: scope_kind(plan/round/scenario/task), kind(hypothesis/observation/question/dissent/evidence/handoff).
- **내용**: from_agent, to_agent(handoff 시 필수), body, payload(JSON 자유 확장).
- **상태**: receipt_acknowledged_at(수신 확인 시각), retired_at(비활성화 시각), retired_reason.
- **타임스탬프**: created_at.

## API 표면

에이전트 노트는 `sdi agent-note` 또는 MCP 도구(`list_agent_notes`, `add_agent_note`)로 관리된다. 활성 노트만 블랙보드 읽기 대상이며 retired 노트는 감사 기록으로만 조회된다.

## 불변식

- **정확한 하나의 닻**: plan/round/scenario/task ID 중 정확히 하나만 설정되어야 한다. 0개 또는 2개 이상이면 거부.
- **scope 일치**: scope_kind와 실제 설정된 닻 ID가 일치해야 한다.
- **handoff에 to_agent 필수(M2)**: kind=handoff면 to_agent가 비어있으면 안 된다.
- **삭제 금지**: 노트는 retire_at으로만 비활성화하며 실제 삭제는 허용하지 않는다.

## 구현 위치 (provenance)

AgentNote 도메인 규칙(닻 검증, handoff 검증)은 `sdi-plugin/crates/core/src/agent_note.rs`에 있다. DB 스키마는 마이그레이션 006에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: receipt_acknowledged_at이 자동으로 설정되는 조건(예: to_agent가 메시지를 읽으면 자동 설정) 또는 수동으로만 설정되는지 확인 필요.
