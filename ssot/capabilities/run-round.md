---
id: capability.run-round
kind: Capability
title: 라운드 실행(생성·활성화·완료)
purpose: "구현 이터레이션(Round)을 열고, 모드(strict-regression / forward-only)와 in-flight 정책을 설정해 활성화하며, 검증 후 완료한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/round.md
  - sdi-plugin/plugin/skills/sdi-round/SKILL.md
relatesTo:
  - { to: concept.round, type: mutates, note: "planning → active → completed 라이프사이클 전체를 관리한다" }
  - { to: concept.round-baseline, type: mutates, note: "strict-regression 모드에서 이전 라운드 판정이 이번 라운드 기준선이 된다" }
  - { to: concept.plan, type: reads, note: "플랜이 active 상태여야 라운드를 만들 수 있다" }
  - { to: concept.scenario, type: reads, note: "활성화 시 needs-verification 목록을 생성해 분해 입력을 제공한다" }
  - { to: concept.task, type: relates-to, note: "라운드 활성화 시 LLM이 태스크를 분해한다" }
  - { to: concept.disruption-review, type: relates-to, note: "DISRUPTION_PENDING 상태면 리뷰 해소 전까지 활성화가 차단된다" }
impacts:
  - concept.round
  - concept.task
  - concept.regression
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

플랜이 승인되면 이터레이션을 라운드로 명시적으로 열고 닫는다. R1은 새 개발, R2 이후는 기본적으로 strict-regression 모드—이전 라운드에서 통과한 시나리오는 자동으로 이월되고, 실패한 시나리오는 이번 라운드에서도 다시 검증해야 한다. 이것이 SDI가 TDD/BDD와 다른 핵심 속성이다. 회귀를 별도의 테스트 스위트가 아니라 라운드 기본값으로 보장한다.

라운드를 만들 때 in-flight 정책도 함께 정한다. 이전 라운드에서 진행 중이던 태스크를 일시 중지할지(기본), 취소할지, 영향 없는 것만 계속할지를 명시한다. 라운드는 한 플랜에 동시에 하나만 active 상태일 수 있다.

## 행위

- `sdi round create <PLAN-ID> <SHORT-CODE> [--mode strict-regression] [--in-flight pause]` 로 planning 상태 라운드를 만든다.
- `sdi round activate <ROUND-ID>` 로 active 로 전환한다. DISRUPTION_PENDING 이면 차단.
- 활성화 직후 LLM이 태스크를 분해한다.
- `sdi round result <ROUND-ID> --scenario <SCN-ID> --result <passing|failing|impacted|retired> --evidence "<ref>"` 로 시나리오별 판정을 기록한다.
- `sdi round complete <ROUND-ID>` 로 닫는다. 전 시나리오가 통과하지 않아도 닫을 수 있다.
- MCP 도구 `start_round` 가 활성화 경로를 대체한다.

## 시스템 흐름

`sdi round create` → planning 상태 생성 → `sdi round activate` → strict-regression 이면 이전 판정 이월 → `scenarios_needing_verification` 반환 → LLM 태스크 분해 → 구현·검증 → 판정 기록(`sdi round result`) → `sdi round complete` → 다음 라운드는 이 판정을 기준선으로 이월.

## 어디에 구현되어 있나

`/round` 슬래시 명령(`plugin/commands/round.md`)과 `sdi-round` 스킬(`plugin/skills/sdi-round/SKILL.md`)이 라운드 전체 절차를 정의한다. MCP `start_round` 도구가 LLM 주도 활성화 경로를 제공한다. 웹 대시보드 BoardView가 라운드 내 태스크 칸반을 표시하고, TimelineView가 라운드 이력을 시각화한다.

## 미확정 (OPEN)

- [ ] OPEN: `sdi round result` 명령에서 `--evidence` ref 가 자동 유효성 검사를 거치는지(단순 문자열 저장인지) 확인 필요.
- [ ] OPEN: MCP `start_round` 가 활성화만 하는지, create까지 포함하는지 도구 스펙 확인 필요.
