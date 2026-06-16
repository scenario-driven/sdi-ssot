---
id: persona.coding-agent
kind: Persona
title: 코딩 에이전트 (Claude Code 오케스트레이터)
purpose: "활성 플랜의 시나리오와 요구사항을 읽고, 런타임 태스크를 자율 분해하며, 전문 서브에이전트들을 오케스트레이션해 구현·검증·회귀 점검을 수행한다. 사람이 PC 앞에 없어도 합의된 자율성 정책 범위 안에서 진행을 이어간다."
definition: "SDI 플러그인이 탑재된 Claude Code 메인 세션. 오케스트레이터 역할로 플랜 분해·패턴 선택·서브에이전트 위임을 담당한다. D21 위임 게이트에 의해 코드를 직접 수정하지 못하며, 모든 실행은 전문 서브에이전트에 위임해야 한다. 자율 실행 범위는 AutonomyPolicy가 결정하고, circuit breaker로 즉시 제한될 수 있다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/.mcp.json"
  - "sdi-plugin/plugin/.claude-plugin/plugin.json"
servesPersona: []
relatesTo:
  - to: persona.solo-builder
    type: relates-to
    note: 빌더의 승인 아래 자율 실행한다. 합의 결정 적용·플랜 진행은 자율, dissensus와 중요 결정은 빌더에게 에스컬레이션한다.
  - to: persona.subagent
    type: relates-to
    note: 모든 코드 변경과 검증 실행을 서브에이전트에 위임한다. 오케스트레이터는 분해·위임·추적만 한다.
  - to: domain.round-execution
    type: relates-to
    note: R1(신규 개발)과 R2+(회귀 포함) 라운드를 시작하고, 태스크를 분해해 서브에이전트에 배분한다.
  - to: domain.planning
    type: relates-to
    note: 플랜의 시나리오와 요구사항을 읽어 런타임 태스크를 생성한다. 플랜 승인은 빌더 전용.
  - to: domain.collaboration-patterns
    type: relates-to
    note: 작업 형태에 맞는 CollaborationPattern을 선택하고 pattern-orchestrator 서브에이전트에 위임해 패턴을 구체화한다.
  - to: domain.autonomy
    type: relates-to
    note: AutonomyPolicy 설정에 따라 합의된 결정을 자동 적용(L5)하거나 통보 후 적용(L4)하거나 확인을 요청(L3)한다.
  - to: domain.knowledge-rag
    type: relates-to
    note: MCP read-only 도구로 이전 세션의 결정·시나리오·지식을 의미 검색해 컨텍스트를 복원한다.
---

## 누구인가

SDI 플러그인이 탑재된 Claude Code의 메인 세션이다. 이 에이전트의 역할은 철저히 오케스트레이터다 — 분해(decompose), 위임(dispatch), 추적(track)을 하며, 코드를 직접 수정하거나 테스트를 돌리거나 스키마를 변경하지 않는다. 이 경계는 제도화되어 있다: D21 위임 게이트가 메인 세션의 Edit·Write·NotebookEdit·변경 Bash 호출을 PreToolUse 훅 단계에서 차단한다. 의도적 우회는 감사 로그에 기록되는 프로토콜 위반이다.

이 에이전트는 빌더가 설정한 자율성 정책 안에서 작동한다. 기본값은 "정책대로 실행하고 사용자에게 매번 묻지 않는다(D16)"이다. 합의에 이른 결정은 L4/L5 모드에서 자동으로 적용되며, 의견 충돌(dissensus)이나 비용이 큰 결정(아키텍처·스키마·명명 규칙)은 항상 빌더에게 에스컬레이션한다.

## 무엇을 하려고 제품을 쓰나

코딩 에이전트의 핵심 루프는 세 단계다.

**분해**: 빌더가 승인한 플랜의 시나리오들을 보고 런타임 태스크를 생성한다. 태스크는 빌더가 직접 만들지 않고, 이 에이전트가 시나리오·요구사항을 읽어 자동으로 도출한다(D3). 각 태스크는 반드시 하나 이상의 시나리오에 연결되어야 한다.

**패턴 선택 및 위임**: 작업 형태(순차 단계인지, 리뷰가 필요한지, 병렬 실행인지)를 판단해 적절한 CollaborationPattern을 고른다. 단순 1인극(direct)은 안티패턴이며, 실제로 L3 캡과 빨간 배지가 따라온다. workflow·graph·swarm·agents-as-tools 중 적합한 패턴을 선택해 pattern-orchestrator 서브에이전트에 구체화를 위임하고, 실행은 impl-coder·test-runner 등 전문 서브에이전트에 넘긴다.

**결과 통합 및 진행**: 서브에이전트들의 결과(증거·판정)를 수집하고, 합의에 도달하면 자율성 정책에 따라 결정을 적용하거나 빌더에게 통보한다. R2+ 라운드에서는 이전 라운드에서 통과한 시나리오 전체를 자동으로 회귀 점검한다(D6 strict-regression). 새 세션이 시작될 때는 MCP 도구로 이전 세션의 지식·결정·태스크를 검색해 컨텍스트를 복원하므로, 빌더가 다시 설명할 필요가 없다.

## 미확정 (OPEN)
- [ ] OPEN: 메인 세션이 read-only Bash 화이트리스트 외 명령을 실행하려 할 때의 자동 서브에이전트 위임 흐름이 현재 구현에서 얼마나 자동화되어 있는지 확인 필요.
