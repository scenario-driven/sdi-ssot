---
id: persona.subagent
kind: Persona
title: 서브에이전트 (SDI 전문 서브에이전트)
purpose: "오케스트레이터(코딩 에이전트)로부터 위임받은 단일 전문 역할을 수행한다 — 코드 구현, 테스트 실행, 결정 협상, 회귀 재검증, 파괴 분석, 패턴 비평, 롤백 실행 등. 각 서브에이전트는 자신의 전문 영역 밖의 일을 하지 않는다."
definition: "SDI plugin/agents/ 에 정의된 specialist 에이전트 유형의 총칭. impl-coder / test-runner / decision-resolver / scenario-decomposer / regression-runner / disruption-analyst / pattern-orchestrator / pattern-critic / schema-architect / reversal-runner / gwt-converter 등 역할별로 분리되어 있다. Claude Code의 Agent 도구로 spawn되며, hookInput.agent_id의 존재가 D21 위임 게이트 통과의 조건이다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
implementedIn:
  - "sdi-plugin/plugin/agents/impl-coder.md"
  - "sdi-plugin/plugin/agents/test-runner.md"
  - "sdi-plugin/plugin/agents/decision-resolver.md"
  - "sdi-plugin/plugin/agents/scenario-decomposer.md"
  - "sdi-plugin/plugin/agents/regression-runner.md"
  - "sdi-plugin/plugin/agents/disruption-analyst.md"
  - "sdi-plugin/plugin/agents/pattern-orchestrator.md"
  - "sdi-plugin/plugin/agents/pattern-critic.md"
  - "sdi-plugin/plugin/agents/reversal-runner.md"
servesPersona: []
relatesTo:
  - to: persona.coding-agent
    type: relates-to
    note: 오케스트레이터로부터 위임받아 실행한다. 완료 후 결과를 AgentNote(핸드오프)로 돌려주거나 다른 서브에이전트에 연결한다.
  - to: domain.round-execution
    type: relates-to
    note: impl-coder와 test-runner가 라운드 내 태스크 구현과 검증의 실질 실행자다.
  - to: domain.decision-negotiation
    type: relates-to
    note: decision-resolver와 schema-architect가 M3 4단계 협상(제안→비평→합의/이견)을 수행한다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: pattern-orchestrator가 패턴을 제안·구체화하고, pattern-critic이 D26/D27 shape 무결성을 감사한다.
  - to: domain.disruption-review
    type: relates-to
    note: disruption-analyst가 변경의 의도치 않은 시나리오 파괴를 탐지하고 사람 확인 게이트를 열어준다.
  - to: domain.agent-coordination
    type: relates-to
    note: AgentNote(M1 블랙보드)와 AgentSpec(M5 자기조직화)을 통해 서브에이전트들이 서로 컨텍스트를 공유하고 핸드오프한다.
---

## 누구인가

서브에이전트는 Claude Code의 `Agent` 도구로 spawn되는 전문 역할 단위다. SDI의 설계 원칙(D13)은 "단일 @main 1인극은 안티패턴"이며 "모든 실행은 전문 서브에이전트에 위임"한다. 이 원칙을 기계적으로 강제하는 장치가 D21 위임 게이트다 — `hookInput.agent_id`가 없는 도구 호출(= 메인 세션의 직접 실행)은 PreToolUse 훅에서 차단된다. 서브에이전트로 spawn되면 이 ID가 생기고 게이트가 열린다.

각 서브에이전트는 명확히 구분된 한 가지 전문 영역을 담당한다. impl-coder는 코드 구현, test-runner는 검증과 증거 수집, decision-resolver는 M3 협상 주도, pattern-orchestrator는 패턴 제안과 구체화, pattern-critic은 패턴 shape 무결성 감사, regression-runner는 이전 라운드 시나리오 재실행, disruption-analyst는 의도치 않은 파괴 탐지, reversal-runner는 결정 롤백 실행, scenario-decomposer는 시나리오에서 태스크 도출, schema-architect는 아키텍처·스키마 비평, gwt-converter는 자연어를 GWT로 변환한다.

AgentSpec 엔티티에는 각 서브에이전트의 이름과 stance(제안자·악마의 변호인·스키마 수호자·성능 검토자·보안 검토자·중립)가 등록된다. Graph 패턴에서 합의 형성 시 (이름, stance) 쌍의 고유성이 2개 이상이어야 한다 — 같은 서브에이전트 종류 두 인스턴스는 판단 다양성을 보장하지 않기 때문이다(D26 sybil 차단).

## 무엇을 하려고 제품을 쓰나

서브에이전트의 목적은 단일하다 — **위임받은 전문 작업을 완수하고 결과를 증거로 남기는 것**.

실행 흐름에서 서브에이전트는 세 가지 방식으로 협력한다.

**단독 실행**: impl-coder가 태스크를 구현하고, test-runner에게 핸드오프 AgentNote를 남긴다. test-runner는 시나리오별 판정(passing/failing/impacted/retired)을 증거 참조(파일:라인 또는 CI URL)와 함께 `sdi task complete`로 제출한다. 증거 없는 완료는 거부된다.

**협상 체인**: decision-resolver가 제안 Decision을 만들면, schema-architect(또는 다른 비평 전문가)가 비평 Decision을 추가한다. 비평이 1개 이상 붙은 후에야 합의 또는 이견 Decision을 발행할 수 있다. 이견이 나오면 빌더에게 에스컬레이션되고 서브에이전트가 임의로 재시도하지 않는다.

**패턴 감사**: pattern-orchestrator가 작업 형태에 맞는 CollaborationPattern을 제안하면, pattern-critic이 D26·D27 shape 기준을 독립적으로 감사한다. 비평을 받은 후 orchestrator가 `pending → active` 전이를 요청한다. 데몬도 이 전이 시점에 shape validation을 다시 검증한다.

## 미확정 (OPEN)
- [ ] OPEN: gwt-converter·schema-architect가 현재 `plugin/agents/` 에 정의 파일이 있으나 실제 에이전트 레지스트리(AgentSpec DB) 초기화 방식은 코드에서 미확인 — 확인 필요.
