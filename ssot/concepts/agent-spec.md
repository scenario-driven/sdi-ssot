---
id: concept.agent-spec
kind: Concept
title: AgentSpec(에이전트 명세)
definition: "Layer 2 전문가 서브에이전트의 등록 정보. 이름, stance(역할 관점), role 설명, system_prompt, 도구 허용 목록, 허용 decision_kind 등을 담는다. 오케스트레이터(Layer 1)가 이 명세를 보고 적합한 전문가를 선택해 디스패치한다."
relatesTo:
  - { to: concept.dispatch, type: relates-to, note: "AgentSpec에 등록된 전문가만 디스패치 대상이 된다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "Graph 패턴의 reviewer, Workflow의 step agent는 AgentSpec의 name으로 참조된다." }
  - { to: concept.consensus, type: relates-to, note: "D26 sybil 차단의 distinctness 기준이 (name, stance) 튜플이다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/agent_spec.rs
  - sdi-plugin/crates/db/src/migrations/006_v04_multi_agent.sql
  - sdi-plugin/crates/db/src/migrations/007_v05_pattern_enforcement.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

AgentSpec(에이전트 명세)는 전문가 서브에이전트를 시스템에 등록하는 레코드다. 오케스트레이터는 이 명세를 보고 어떤 전문가를 어떤 작업에 배정할지 결정한다.

**Stance(입장)**는 에이전트가 어떤 관점에서 작업하는지를 나타낸다. 같은 이름의 에이전트도 stance가 다르면 D26 sybil 차단에서 구별된 것으로 인정된다.

| Stance | 의미 |
|--------|------|
| proposer | 제안자 |
| devil_advocate | 반론자 |
| schema_guardian | 스키마 수호자 |
| performance_reviewer | 성능 검토자 |
| security_reviewer | 보안 검토자 |
| neutral | 중립(기본값) |

유효한 에이전트 이름은 두 집합으로 제한된다(M5).

**8개 기본 전문가**: gwt-converter, scenario-decomposer, impl-coder, test-runner, regression-runner, disruption-analyst, decision-resolver, schema-architect(모두 neutral stance 기본).

**3개 v0.5 메타 전문가**: pattern-orchestrator(proposer), pattern-critic(devil_advocate), reversal-runner(neutral).

이 목록 외의 이름으로 AgentSpec을 생성하면 도메인 레벨에서 거부된다. 동적으로 새 역할을 발명하는 것은 M5에서 금지한다.

AgentSpec의 lifecycle은 Active/Expired 두 상태다. Expired 행은 감사 기록으로 보존되나 dispatch 후보에서 제외된다.

`instance_count`는 하나의 AgentSpec이 동시에 스폰될 수 있는 최대 인스턴스 수(0~16)다.

## 엔티티 (DB)

AgentSpec 한 건은 다음 정보를 보존한다.

- **식별**: name, stance.
- **설명**: role(역할 설명), system_prompt(에이전트 프롬프트).
- **제어**: instance_count(0~16), tool_allowlist_json(도구 허용 목록), decision_kinds_json(허용 decision kind 목록), blast_radius_rules_json(blast_radius 조정 규칙).
- **출처**: created_by, origin_plan_id(플랜 범위 에이전트인 경우).
- **상태**: status(active/expired), expires_at(optional).
- **타임스탬프**: 생성, 수정 시각.

## API 표면

AgentSpec은 시스템 시작 시 seed 데이터로 기본값이 주입된다. 사용자나 에이전트도 명세를 조회하고 일부 속성을 수정할 수 있다.

## 불변식

- **등록된 이름만 허용(M5)**: STOCK_AGENTS + STOCK_META_AGENTS 외 이름 불가.
- **instance_count 0~16(M5)**: 최대 16개 인스턴스. 범위 외 값 거부.
- **sybil 차단(D26)**: Graph 패턴 consensus에서 (name, stance) 튜플이 구별되어야 한다.

## 구현 위치 (provenance)

AgentSpec 도메인 규칙(`validate_name`, `validate_instance_count`, STOCK_AGENTS, STOCK_META_AGENTS)은 `sdi-plugin/crates/core/src/agent_spec.rs`에 있다. DB 스키마는 마이그레이션 006(핵심), 007(stance, blast_radius_rules_json 등 추가)에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: `decision_kinds_json`의 `["*"]`가 모든 kind를 허용한다고 주석에 있는데, 정확히 어떤 검사 로직과 연동되는지 daemon 레벨 확인 필요.
