---
id: capability.manage-plan
kind: Capability
title: 플랜 생성·승인·완료
purpose: "사용자가 구현 의도를 플랜으로 만들고, 시나리오가 준비되면 승인해 라운드를 시작할 수 있도록 하며, 완료 시 닫는다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/plan.md
  - sdi-plugin/plugin/commands/plan.md
relatesTo:
  - { to: concept.plan, type: mutates, note: "draft → active → completed 전체 라이프사이클을 관리한다" }
  - { to: concept.scenario, type: reads, note: "승인 게이트에서 confirmed 시나리오 ≥1 를 요구한다" }
  - { to: concept.round, type: relates-to, note: "승인된(active) 플랜에서만 라운드를 만들고 활성화할 수 있다" }
  - { to: concept.project, type: reads, note: "플랜은 반드시 한 프로젝트에 귀속된다. 프로젝트당 active 플랜은 하나" }
  - { to: concept.decision, type: relates-to, note: "플랜 본문 변경의 이유는 decide로 기록한다" }
impacts:
  - concept.plan
  - concept.round
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

무엇을 만들 것인지를 플랜으로 선언한다. 플랜은 승인의 단위다. 초안(draft) 상태의 플랜에 시나리오와 요구사항을 추가하고, 검증 가능한 동작이 하나 이상 확정되면 승인(approve)해 active 상태로 올린다. 이 순간부터 라운드를 만들어 실제 구현·검증을 시작할 수 있다. 일이 끝나면 complete로 닫는다.

승인 게이트(D8)가 핵심이다. "confirmed 시나리오가 하나도 없는 플랜"은 승인할 수 없다. 이 규칙이 "무엇을 검증할지 결정하기 전에는 구현을 시작하지 않는다"는 SDI의 철학을 강제한다. 태스크 수는 게이트에 영향을 주지 않는다—태스크는 승인 후 LLM이 분해하는 런타임 산출물이다.

## 행위

- `sdi plan create <PROJECT-ID> <SHORT-CODE> "<title>"` 으로 draft 플랜을 만든다.
- `sdi plan approve <PLAN-ID>` 로 active 로 전환한다. confirmed 시나리오 없으면 `SCENARIOS_REQUIRED`.
- `sdi plan complete <PLAN-ID>` 로 닫는다. 시나리오가 전부 통과하지 않아도 닫을 수 있다.
- `sdi plan active <PROJECT-ID>` 로 현재 활성 플랜을 조회한다.
- `sdi plan view <PLAN-ID>` 로 상세를 확인한다.
- 플랜 본문은 스냅샷 방식—덮어쓰기. 변경 이유는 `/decide`에 기록.

## 시스템 흐름

프로젝트 등록 → `/plan create` 로 draft 플랜 생성 → `/scenario`, `/req` 로 내용 채움 → `/plan approve` 게이트 통과(confirmed 시나리오 ≥1) → 플랜이 active, 라운드 생성 가능 → 라운드 완료 후 `/plan complete` 으로 닫기. 프로젝트당 active 플랜은 동시에 하나만 허용—이전 플랜을 완료하거나 draft 상태여야 한다.

## 어디에 구현되어 있나

`/plan` 슬래시 명령(`plugin/commands/plan.md`)과 `plan` 스킬(`plugin/commands/plan.md`)이 전체 라이프사이클 절차를 정의한다. 대시보드 웹(`plugin/web/src/views/`)의 SummaryView와 BoardView가 플랜 상태를 시각화한다. 데몬 HTTP API가 상태 전환 및 게이트 검증을 수행한다.

## 미확정 (OPEN)

- [ ] OPEN: 웹 대시보드에서 플랜 생성·승인 UI가 구현되어 있는지(현재 CLI 전용인지) 확인 필요.
