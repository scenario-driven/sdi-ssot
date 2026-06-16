---
id: capability.author-scenario
kind: Capability
title: GWT 시나리오 작성
purpose: "사람이나 LLM 에이전트가 시스템이 반드시 보여야 할 동작을 Given/When/Then 형식의 원자적 시나리오로 작성해 플랜에 등록한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/scenario.md
  - sdi-plugin/plugin/skills/sdi-scenario/SKILL.md
relatesTo:
  - { to: concept.scenario, type: mutates, note: "시나리오를 새로 만들거나 confirmed 상태로 전환한다" }
  - { to: concept.given-when-then, type: mutates, note: "G/W/T 세 절이 모두 비어 있지 않아야 하며 데몬이 이를 강제한다(GWT_EMPTY)" }
  - { to: concept.plan, type: reads, note: "시나리오는 반드시 특정 플랜 아래에 귀속된다" }
  - { to: concept.scenario-id, type: mutates, note: "짧은 코드(예: SC-12)를 부여해 플랜 내에서 고유 식별한다" }
  - { to: concept.ticket-number, type: mutates, note: "short_code 가 사람이 읽는 티켓 번호 역할을 한다" }
impacts:
  - concept.scenario
  - concept.plan
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

시스템이 어떻게 동작해야 하는지를 자연어로 말하면 LLM이 그것을 세 개의 절—주어진 상태(Given), 사용자 행동(When), 기대 결과(Then)—로 정리해 플랜에 등록한다. 등록된 시나리오는 라운드 검증의 단위가 되고, 이후 회귀 검증에서도 자동으로 재확인된다. 사람이 Gherkin 문법을 배울 필요가 없고, 스텝 글루(step glue)를 작성할 필요도 없다. LLM이 자연어를 직접 읽고 검증한다.

시나리오는 한 번에 하나의 검증 가능한 사실만 담아야 한다. "그리고", "또는", 여러 개의 Then이 한 시나리오에 섞이면 분리해야 한다. 이 원자성이 라운드 간 세밀한 회귀 신호를 만든다.

## 행위

- 사람이 자유로운 자연어로 동작을 묘사하면 LLM이 G/W/T 형태로 정규화한다.
- `/scenario` 명령 또는 `sdi-scenario` 스킬을 통해 `sdi scenario create` CLI 호출로 데몬에 저장한다.
- `--confirmed` 플래그를 주면 즉시 confirmed 상태로, 생략하면 draft 상태로 등록된다.
- 플랜 ID를 생략하면 현재 cwd 기준 활성 플랜을 자동으로 조회한다.
- 하나의 설명에서 여러 동작이 나오면 시나리오를 분리해 각각 등록한다.

## 시스템 흐름

사용자가 동작을 묘사하면 → `sdi-scenario` 스킬이 G/W/T 절로 정규화 → `sdi scenario create <PLAN-ID> <SHORT-CODE> --given … --when … --then …` CLI 호출 → 데몬이 세 절 비어있지 않음을 검증 → SQLite에 저장, 반환된 시나리오 ID 사용자에게 고지 → 플랜 approve 게이트(`≥1 confirmed`)에 반영.

MCP 경유 시에는 `add_scenario` 도구가 같은 등록 경로를 수행한다.

## 어디에 구현되어 있나

`/scenario` 슬래시 명령(`plugin/commands/scenario.md`)이 GWT 파라미터를 받아 CLI를 호출하는 절차를 정의한다. `sdi-scenario` 스킬(`plugin/skills/sdi-scenario/SKILL.md`)은 자연어를 G/W/T 형식으로 정규화하는 LLM 측 권위자다. MCP 도구 `add_scenario`는 LLM이 직접 구동할 때 동일 등록 경로의 대안이다.

## 미확정 (OPEN)

- [ ] OPEN: draft 상태 시나리오가 confirmed 로 전환되는 명시적 명령 경로(별도 subcommand인지, `--confirmed` 재전달인지) 코드 확인 필요.
