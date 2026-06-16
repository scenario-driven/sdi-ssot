---
id: capability.orchestrate-pattern
kind: Capability
title: 협업 패턴 선택 및 오케스트레이션
purpose: "작업 형태에 맞는 다중 에이전트 협업 패턴을 선택해 활성화하고, 그 안에서 태스크·결정·라운드가 생산되도록 오케스트레이션한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/commands/pattern.md
  - sdi-plugin/plugin/commands/pattern.md
relatesTo:
  - { to: concept.collaboration-pattern, type: mutates, note: "패턴을 생성하고 pending → active → converged/dissensus/aborted 로 전환한다(D22)" }
  - { to: concept.dispatch, type: mutates, note: "활성 패턴이 작업 엔티티 생산 방식을 결정해 디스패치 구조를 정의한다" }
  - { to: concept.task, type: relates-to, note: "태스크 생성 시 produced_via_pattern_id 가 NOT NULL 이어야 한다(D23)" }
  - { to: concept.decision, type: relates-to, note: "결정도 패턴을 통해 생산된다; graph 패턴이 합의 협상의 구조를 제공한다" }
  - { to: concept.autonomy-policy, type: reads, note: "패턴 종류마다 기본 자율성 모드가 달라진다(D25): workflow/graph=L5, swarm/agents-as-tools=L4, direct=L3 강제" }
  - { to: concept.pattern-lifecycle, type: mutates, note: "패턴 라이프사이클(pending→active→terminal)을 관리한다" }
impacts:
  - concept.collaboration-pattern
  - concept.task
  - concept.autonomy-policy
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

라운드가 활성화된 후 태스크를 분해하기 전에, 어떤 방식으로 에이전트들이 협력할지를 결정한다. 이것이 D13의 핵심—"다중 에이전트 오케스트레이션이 부산물이 아니라 본체"라는 선언이다. 작업 형태에 따라 네 가지 패턴 중 하나를 고른다:

- **workflow**: 순차 핸드오프(초안 → 비판 → 검증). 단계가 ≥2 이어야 한다.
- **graph**: 서로 다른 입장의 심의자가 ≥2 명 참여하는 합의. 시빌 공격 차단.
- **swarm**: ≥2 명의 전문가가 병렬로 태스크를 처리. 팬아웃 구조.
- **agents-as-tools**: 호출자 에이전트가 피호출 전문가를 도구처럼 사용.

`direct`는 진정한 단독 작업임을 명시하는 방식이고, 그 패널티로 L3 상한을 강제한다. 기본값으로 떨어지는 것이 아니라 명시적으로 선택해야 한다.

## 행위

- `sdi pattern create` 로 원하는 종류와 형태 매니페스트를 담아 패턴을 만든다.
- `sdi pattern transition <PAT-ID> --to active` 로 활성화한다. 이 시점에 D26 형태 게이트 통과.
- `sdi pattern tree --plan <PLAN-ID>` 로 중첩 패턴 DAG를 조회한다.
- `sdi pattern abort <PAT-ID>` 로 중단한다.
- 태스크 생성 시 `--produced-via-pattern <PAT-ID>` 로 바인딩한다.
- 깊이 제한(기본 3)을 초과하는 중첩은 데몬이 거부한다(D24).

## 시스템 흐름

라운드 활성화 → pattern-orchestrator 에이전트 스폰 → 작업 집합 분석 후 패턴 종류 결정 → `sdi pattern create … --kind <K>` → pattern-critic 에이전트가 D26 형태 검증 → `sdi pattern transition --to active` → 태스크 분해 시 `--produced-via-pattern <PAT-ID>` 바인딩 → 패턴이 converged / dissensus / aborted 로 종결.

## 어디에 구현되어 있나

`/pattern` 슬래시 명령(`plugin/commands/pattern.md`)이 생성·전환·조회 절차를 정의한다. `pattern` 스킬(`plugin/commands/pattern.md`)이 선택 기준을 안내한다. 데몬 HTTP API가 형태 게이트, DAG 사이클 검사, depth cap을 강제한다. 웹 대시보드 PatternsView가 활성 패턴 상태를 시각화한다.

## 미확정 (OPEN)

- [ ] OPEN: PatternsView 가 패턴 중첩 DAG를 트리로 렌더링하는지, 아니면 단순 리스트인지 웹 코드 확인 필요.
