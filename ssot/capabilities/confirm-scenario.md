---
id: capability.confirm-scenario
kind: Capability
title: 시나리오 확정(Confirm)
purpose: "draft 상태의 시나리오를 사람이나 에이전트가 검토하고 confirmed 로 전환해 플랜 승인 게이트를 충족시킨다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/scenario.md
  - sdi-plugin/plugin/skills/sdi-scenario/SKILL.md
relatesTo:
  - { to: concept.scenario, type: mutates, note: "draft → confirmed 상태 전환이 이 역량의 핵심 변환이다" }
  - { to: concept.plan, type: reads, note: "플랜 승인 게이트(≥1 confirmed)를 충족하기 위해 confirm 이 필요하다" }
  - { to: concept.round, type: relates-to, note: "confirmed 시나리오만 라운드 검증 대상에 포함된다" }
impacts:
  - concept.scenario
  - concept.plan
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

시나리오를 작성한 뒤 내용이 확정됐으면 confirmed 상태로 바꾼다. 이 전환이 의미하는 것은 "이 동작은 실제로 검증해야 하는 요구사항이다"라는 사람의 명시적 동의다. 플랜은 confirmed 시나리오가 하나 이상 있어야 승인할 수 있고, 라운드를 돌릴 수 있다.

draft는 초안 단계—아직 확정되지 않은 가능성이다. confirmed는 검증 대상—라운드가 이 시나리오를 반드시 확인해야 한다는 선언이다. 사람이 이 전환을 통제함으로써 "무엇을 검증할 것인가"에 대한 결정권을 유지한다.

## 행위

- `sdi scenario create … --confirmed` 로 처음부터 confirmed 상태로 등록한다.
- 이미 등록된 draft 시나리오를 별도 subcommand로 confirmed 상태로 전환한다.
- 플랜 ID와 시나리오 ID를 지정해 특정 시나리오만 선택적으로 확정한다.
- 확정 후 플랜 approve 게이트 충족 여부가 달라진다.

## 시스템 흐름

draft 시나리오 존재 → 사람이 내용 검토 → `--confirmed` 포함 create 또는 confirm subcommand 호출 → 데몬이 상태를 `confirmed`로 변경 → 플랜의 "confirmed 시나리오 수" 카운트가 증가 → `sdi plan approve` 가 SCENARIOS_REQUIRED 없이 통과 가능.

## 어디에 구현되어 있나

`/scenario` 슬래시 명령(`plugin/commands/scenario.md`)이 `--confirmed` 플래그 처리를 정의한다. `sdi-scenario` 스킬은 작성 시 즉시 confirmed 로 넘길지, draft 로 둘지 판단하는 기준을 제공한다. 데몬 HTTP API가 상태 전환을 실제 수행한다.

## 미확정 (OPEN)

- [ ] OPEN: draft → confirmed 전환을 위한 별도 `sdi scenario confirm <SCN-ID>` subcommand 존재 여부 코드 확인 필요.
