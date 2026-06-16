---
id: component.agent-pattern-critic
kind: SystemComponent
title: pattern-critic 서브에이전트
purpose: pattern-orchestrator가 제안한 CollaborationPattern 후보를 사전 심사해 시빌 그래프, 1단계 워크플로우, 단일 인스턴스 스웜 등 D26·D27 게이트를 통과하지 못할 형상을 데몬 API 호출 전에 걸러내는 비평 전문 에이전트다.
realizedBy:
  - domain.collaboration-patterns
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/pattern-critic.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-pattern-orchestrator
    type: preceded-by
    note: orchestrator가 패턴을 생성(pending)하면 critic이 전환 전에 형상을 심사한다
  - to: domain.collaboration-patterns
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

pattern-critic은 패턴 심사 전문 에이전트다. 입장(stance)은 `devil_advocate`로 고정된다. pattern-orchestrator가 `pending` 상태의 패턴을 생성한 뒤, `pending → active` 전환을 시도하기 전에 이 에이전트에게 형상 검토를 위임한다. 데몬의 D26·D27 게이트가 내보내는 단순한 HTTP 400 대신 풍부한 맥락과 함께 문제를 사전에 발견하는 것이 목적이다.

심사 기준은 패턴 종류마다 다르다. **workflow** 패턴은 단계가 2개 미만이면 가짜 패턴(fake pattern)으로 판정한다 — 1단계 workflow는 `direct`의 L3 상한을 우회하는 수단이 될 수 없다. **graph** 패턴은 합의에 필요한 `(AgentSpec.name, AgentSpec.stance)` 튜플이 2개 이상 다른지 확인한다 — 동일 에이전트가 다른 이름으로 반복 등장하는 시빌 그래프를 차단한다. **swarm** 패턴은 fan_out ≥ 2이고 자기 스폰이 아닌지 검사한다. **agents-as-tools** 패턴은 피어가 1개 이상 등록되어 있는지 확인한다.

심사 통과 시 orchestrator에게 `pending → active` 전환을 진행하도록 보고한다. 실패 시 무엇이 문제인지 구체적으로 기술하고 orchestrator가 패턴을 재설계하도록 돌려보낸다.

## 경계와 의존

Bash·Read·Grep·Glob 도구로 패턴 정보를 읽고 형상을 검사한다. 패턴 행을 직접 수정하지 않으며, `sdi pattern show` CLI 명령으로 패턴 상태를 읽어 판단한다. 결과를 Skill·SendMessage로 orchestrator에 전달한다.

## 통신 패턴

pattern-orchestrator가 critic을 스폰하거나 SendMessage로 심사를 요청한다. critic은 통과/실패 판정과 근거를 반환하고 종료한다. graph 패턴 심사처럼 비평 역할 자체가 이 에이전트의 목적이므로 독립 전문 에이전트로 분리된다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/pattern-critic.md`)로 구성된다. 심사 체크리스트·종류별 거부 조건·워크플로우가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: pattern-critic이 발견한 형상 오류를 orchestrator에게 전달하는 채널(SendMessage vs agent-note vs 반환 메시지)이 구체적으로 명세될 필요가 있다.
