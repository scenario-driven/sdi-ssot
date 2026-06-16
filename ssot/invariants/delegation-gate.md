---
id: invariant.delegation-gate
kind: Invariant
title: 메인 세션은 실행 도구를 직접 사용할 수 없다 — specialist sub-agent에만 위임된다
definition: "orchestrator(메인 Claude Code 세션)는 Edit·Write·NotebookEdit·mutating Bash 같은 실행 도구를 직접 호출할 수 없다. 이 도구들은 Agent 도구로 spawn된 specialist sub-agent에서만 허용된다. hookInput.agent_id 부재가 메인 세션의 신호이며, PreToolUse 훅이 이 조건에서 실행 도구를 차단한다."
governs:
  - domain.agent-coordination
  - domain.collaboration-patterns
  - domain.autonomy
  - concept.collaboration-pattern
  - concept.dispatch
  - concept.agent-note
implementedIn:
  - "sdi-plugin/plugin/adapters/claude/pre-tool-use.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
  - "sdi-plugin/plugin/hooks/hooks.json"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI에서 orchestrator(메인 세션)와 specialist sub-agent의 역할 분리는 협업 아키텍처의 근간이다(D13, D21). 메인 세션은 계획(plan)·분해(decompose)·위임(dispatch)만 수행한다. 실제 코드 변경, 파일 쓰기, mutating 쉘 명령, 외부 부수효과는 반드시 Agent 도구로 spawn된 specialist sub-agent가 수행해야 한다.

이 분리는 문서 규약에 그치지 않고 PreToolUse 훅에 의해 런타임에 기계적으로 강제된다. Claude Code 공식 훅 계약에 따르면 `hookInput.agent_id`는 Agent 도구로 spawn된 sub-agent의 훅 호출에서만 존재한다. 메인 세션의 훅 호출에는 이 필드가 없다. 훅이 이 신호를 감지하여 메인 세션의 실행 도구 호출을 거부한다.

read-only 도구(`Read`, `Grep`, `Glob`, `WebSearch`, `WebFetch`, `Agent`, `TaskCreate/Update/List`, `SendMessage`, MCP read 도구)와 read-only Bash(status/log/diff/typecheck/lint/cargo check 등 화이트리스트 매칭)는 메인 세션에서도 허용된다. 메인이 분해·위임을 수행하는 데 필요한 도구들이다.

등록되지 않은 `agent_type`으로 spawn된 sub-agent(rogue agent)의 실행 도구도 차단된다. AgentSpec에 등록된 specialist만 실행 도구를 사용할 수 있다.

비상 우회는 `sdi bypass arm --reason "<사유>"` 명령으로 단 하나의 도구 호출에 한해 해제할 수 있으며, 모든 우회는 감사 로그에 `pre_tool_use_delegation_bypass`로 기록된다. 우회 남용은 protocol violation이다.

## 깨지면 무슨 일이 일어나나

메인 세션이 실행 도구를 직접 사용할 수 있다면 D13의 "multi-agent orchestration이 본체"라는 원칙이 런타임 수준에서 무의미해진다. 단일 @main 1인극(solo flow)이 사실상 L5 자율 실행과 동일한 권한을 갖게 되며, 패턴 검증(D26/D27), consensus 게이트(D20), 회귀 안전망이 우회된다. 특히 메인이 직접 코드를 쓰면 어떤 시나리오에 대한 구현인지 연결이 없어 Task evidence 추적이 불가능해진다.

## 코드에서 어떻게 강제되나

`plugin/hooks/hooks.json`의 PreToolUse 매처가 `Edit|Write|MultiEdit|Bash|NotebookEdit|Agent|Task|TeamCreate|SendMessage`를 대상으로 `pre-tool-use.cjs`를 실행한다. `plugin/adapters/shared/sdi-hooks.cjs`의 핵심 로직이 `hookInput.agent_id` 존재 여부를 검사한다. 부재(=메인 세션) + 실행 도구 조합이면 `permissionDecision: deny`를 반환한다. `agent_id` 존재 + `agent_type`이 AgentSpec registry에 있으면 통과, registry에 없으면 `rogue-specialist` 코드로 거부한다. `plugin/tests/delegation.test.cjs`가 이 로직을 검증한다. HOOK_ENFORCEMENT.md의 신뢰 경계 역할을 한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D21 Mandatory Delegation Gate 결정 엔트리) 연결 필요
