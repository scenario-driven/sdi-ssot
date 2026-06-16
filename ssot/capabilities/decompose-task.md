---
id: capability.decompose-task
kind: Capability
title: 태스크 자동 분해
purpose: "라운드 활성화 시 LLM이 검증 필요 시나리오와 요구사항을 읽어 런타임 태스크로 분해하고, 각 태스크를 협업 패턴에 연결한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-round/SKILL.md
relatesTo:
  - { to: concept.task, type: mutates, note: "태스크는 라운드 활성화 후 LLM이 분해하는 런타임 산출물이다(D3)" }
  - { to: concept.scenario, type: reads, note: "needs-verification 목록의 시나리오가 분해 입력이 된다" }
  - { to: concept.round, type: reads, note: "활성 라운드 컨텍스트(mode, 이월 판정) 안에서 분해가 일어난다" }
  - { to: concept.collaboration-pattern, type: calls, note: "태스크 생성 전 패턴 결정이 선행돼야 한다(D13); 패턴 없으면 direct sentinel 적용" }
  - { to: concept.dispatch, type: relates-to, note: "분해된 태스크는 패턴에 따라 전문가 서브에이전트에 디스패치된다" }
impacts:
  - concept.task
  - concept.round
  - concept.collaboration-pattern
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

라운드를 활성화하면 SDI는 이 라운드에서 검증해야 할 시나리오 목록을 내놓는다. 사람이 태스크를 직접 작성하지 않는다. LLM이 그 목록을 읽고 각 시나리오마다 하나 이상의 태스크를 만들어 연결한다. 이것이 D3 결정—"태스크는 런타임 산출물"—의 의미다. 태스크는 시나리오가 통과했음을 증명하기 위한 작업 단위이지, 사람이 사전에 기획하는 작업 목록이 아니다.

분해 전에 어떤 협업 패턴으로 작업을 배분할지 결정해야 한다(D13). 순차 핸드오프(workflow), 다수 전문가 병렬 실행(swarm), 심의 기반 합의(graph), 도구로서의 에이전트(agents-as-tools) 중 작업 형태에 맞는 패턴을 선택하고 active 상태로 만든 뒤 각 태스크를 그 패턴에 연결한다. 이 단계를 건너뛰면 데몬이 `direct` 센티넬을 자동으로 채워 L3 상한이 적용된다.

## 행위

- 라운드 활성화 후 `scenarios_needing_verification` 목록을 수신한다.
- pattern-orchestrator 전문 에이전트를 실행해 협업 패턴 종류와 구조를 결정한다.
- `sdi task create <ROUND-ID> <SHORT-CODE> "<desc>" --scenario <SCN-ID> --produced-via-pattern <PAT-ID>` 로 각 태스크를 생성한다.
- 태스크는 반드시 해당 라운드의 needs-verification 시나리오에 연결되어야 한다.
- 패턴 참조가 `active` 가 아니거나 다른 플랜/라운드에 속하면 데몬이 거부한다.

## 시스템 흐름

`sdi round activate <ROUND-ID>` → 데몬이 `scenarios_needing_verification` 반환 → pattern-orchestrator 실행으로 협업 패턴 결정·활성화 → `sdi task create` 반복 호출로 각 시나리오에 태스크 연결 → 이후 impl-coder, test-runner 등 전문가 에이전트가 각 태스크를 실행 → 증거 기록 후 `sdi task done`.

## 어디에 구현되어 있나

분해 절차는 `sdi-round` 스킬(`plugin/skills/sdi-round/SKILL.md`)의 "4. Decompose under the chosen pattern" 단계에 정의된다. `sdi task create` CLI가 실제 생성을 수행하고, 데몬이 패턴 유효성과 시나리오 귀속 관계를 검증한다. 웹 대시보드 BoardView는 분해된 태스크의 칸반 상태를 시각화한다.

## 미확정 (OPEN)

- [ ] OPEN: 분해 이전 패턴 결정 단계에서 pattern-orchestrator 에이전트 스폰이 PreToolUse 훅에 의해 어떻게 강제되는지 정확한 체인 확인 필요.
