---
id: integration.claude-code-subagents
kind: Integration
title: Claude Code 서브에이전트 스페셜리스트 연동
purpose: 오케스트레이터(메인 세션)가 실행을 직접 하지 않고 스페셜리스트 서브에이전트에 위임하도록 — SDI의 "multi-agent orchestration이 본체"(D13) 원칙을 Claude Code의 `Agent` 도구 위에서 작동시킨다. 스페셜리스트는 시나리오 분해·구현·검증·회귀·결정 해소·패턴 선택·롤백 등 역할별로 분리되어 있다.
definition: "SDI 플러그인이 11개 스페셜리스트 에이전트 정의 파일(`plugin/agents/*.md`)을 Claude Code의 에이전트 검색 경로에 놓는다. PreToolUse 훅이 에이전트 `agent_id` 유무와 등록된 `agent_type` 일치 여부를 확인해 미등록 에이전트의 실행 도구 사용을 차단한다(D21 Delegation Gate)."
integratesWith: []
implementedIn:
  - sdi-plugin/plugin/agents/scenario-decomposer.md
  - sdi-plugin/plugin/agents/gwt-converter.md
  - sdi-plugin/plugin/agents/impl-coder.md
  - sdi-plugin/plugin/agents/test-runner.md
  - sdi-plugin/plugin/agents/regression-runner.md
  - sdi-plugin/plugin/agents/disruption-analyst.md
  - sdi-plugin/plugin/agents/schema-architect.md
  - sdi-plugin/plugin/agents/decision-resolver.md
  - sdi-plugin/plugin/agents/pattern-orchestrator.md
  - sdi-plugin/plugin/agents/pattern-critic.md
  - sdi-plugin/plugin/agents/reversal-runner.md
  - sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs
impacts:
  - domain.agent-coordination
  - domain.collaboration-patterns
  - domain.autonomy
  - domain.round-execution
  - domain.disruption-review
  - domain.decision-negotiation
relatesTo:
  - to: domain.agent-coordination
    type: realizes
    note: 11개 스페셜리스트 역할 분리가 에이전트 조율 도메인의 실체다
  - to: domain.collaboration-patterns
    type: realizes
    note: pattern-orchestrator·pattern-critic 에이전트가 협업 패턴 도메인을 직접 실행한다
  - to: domain.autonomy
    type: realizes
    note: Delegation Gate(D21)가 AutonomyPolicy의 실행 통제 면이다
  - to: domain.decision-negotiation
    type: realizes
    note: decision-resolver가 4단계 협상 흐름(M3)을 에이전트로 구체화한다
  - to: domain.disruption-review
    type: realizes
    note: disruption-analyst가 시나리오 변경 파급 분류를 전담한다
  - to: domain.round-execution
    type: realizes
    note: impl-coder·test-runner·regression-runner가 라운드 실행의 핵심 전담자다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 Claude Code의 서브에이전트 실행 인프라다. Claude Code는 `Agent` 도구를 호출할 때 `agent_type` 이름을 세 경로에서 찾는다: 프로젝트의 `.claude/agents/`, 사용자 홈의 `~/.claude/agents/`, 플러그인의 `agents/`. SDI는 세 번째 경로에 11개 스페셜리스트 정의 파일을 두어 에이전트 레지스트리에 참여한다.

각 스페셜리스트의 역할은 다음과 같다.

- **scenario-decomposer**: 플랜 의도를 GWT 시나리오로 분해한다. 아직 시나리오가 없는 플랜에서 LLM이 초안을 제안할 때 사용한다.
- **gwt-converter**: 자유 형식 자연어 요구를 D5 준수 Given/When/Then 형식으로 변환한다.
- **impl-coder**: 시나리오가 연결된 태스크의 코드를 구현한다. 시나리오가 스펙이므로 반드시 먼저 읽고 구현한다.
- **test-runner**: 구현 변경에 대해 테스트를 실행하고 증거를 구조화된 형식으로 수집한다.
- **regression-runner**: R2+ 라운드에서 이전 라운드에서 통과한 시나리오를 재실행해 회귀를 확인한다(D6 strict-regression 기본값).
- **disruption-analyst**: 시나리오·요구사항·결정 변경이 기존 시나리오에 미치는 파급을 분류한다(D9 needs-review 기본값).
- **schema-architect**: 스키마·마이그레이션·퍼블릭 API에 닿는 결정을 담당한다. D17에 의해 L4 강제 — 반드시 결정 제안자가 아니라 비평자로만 참여한다.
- **decision-resolver**: M3 4단계 협상(제안→비평→합의 or 분쟁 에스컬레이션)을 구동하고 결정 행을 생성한다(D20).
- **pattern-orchestrator**: 새 작업 엔티티에 CollaborationPattern을 선택하고 활성화한다(D22·D27).
- **pattern-critic**: Graph 패턴의 Sybil 차단을 위해 독립적인 `(name, stance)` 튜플을 제공하는 두 번째 검토자 역할을 한다(D26).
- **reversal-runner**: 롤백이 결정된 Decision에 대해 `reversal_plan`에 명시된 역조작(마이그레이션 SQL·git revert·파일 스냅샷 복원 등)을 실행하고 append-only Decision 행으로 기록한다(D28).

## 구현 위치 (provenance)

각 에이전트 정의는 `plugin/agents/<name>.md` 파일이다. YAML frontmatter에 이름·설명·허용 도구·모델 지시가 있고, 본문은 해당 스페셜리스트의 작업 원칙과 워크플로우를 자연어로 기술한다.

PreToolUse 훅의 Delegation Gate는 `plugin/adapters/shared/sdi-hooks.cjs` 안에서, `hookInput.agent_id`가 없는 호출(메인 세션)을 실행 도구에서 차단하고, `agent_id`가 있는 호출(서브에이전트)은 등록된 에이전트 이름과 `agent_type`을 대조해 미등록이면 차단한다. 에이전트 레지스트리는 세 루트 경로의 `.md` 파일 이름을 런타임에 스캔하며 디렉토리 mtime을 기준으로 캐시된다.

## 불변식

- 메인 세션은 `Edit`·`Write`·`MultiEdit`·`NotebookEdit`·mutating `Bash`를 직접 실행할 수 없다(D21). 유일한 합법 경로는 스페셜리스트 서브에이전트에 위임하는 것이다.
- 등록되지 않은 `agent_type`으로 spawn된 서브에이전트도 실행 도구를 사용할 수 없다(rogue agent 방지).
- `schema-architect`는 결정의 원 제안자가 될 수 없다. D17 L4 강제 대상 결정 종류(architecture·schema·naming-canonical)에서는 비평자 역할만 허용된다.
- D20에 의해 단일 에이전트 결정은 L3(항상 사용자 확인) 상한을 넘을 수 없다. `decision-resolver`가 다단계 협상을 통해 multi-agent consensus를 이끌어야 L4/L5 적용이 가능하다.

## 영향 범위

이 연동이 끊기거나 에이전트 파일이 손상되면 SDI의 모든 작업 실행이 오케스트레이터에게 막힌다. 오케스트레이터는 계획·분해·위임만 가능하므로, 실행 스페셜리스트 없이는 사실상 아무 코드도 쓸 수 없다. 패턴 선택, 결정 협상, 롤백도 마찬가지로 멈춘다. 즉 이 연동은 SDI의 multi-agent 본체(D13)를 물리적으로 구현하는 핵심 표면이다.

## 미확정 (OPEN)
- [ ] OPEN: 서브에이전트 `agent_type`과 에이전트 파일 이름의 정확한 매핑 규칙(플러그인 네임스페이스 접두어 처리 등) 추가 확인 필요.
- [ ] OPEN: 라운드 실행 중 여러 스페셜리스트가 병렬로 실행될 때 리소스 클레임(D29)과의 상호작용 흐름 명시 필요.
