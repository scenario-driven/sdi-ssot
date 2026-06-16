---
id: decision.pattern-enforcement
kind: Decision
title: CollaborationPattern을 7번째 1등 시민으로 두고 모든 작업 엔티티의 생성 출처를 강제 기록한다
purpose: "에이전트 협업 패턴을 코드 상수나 관례로 둘지, 영속 DB 엔티티로 두어 각 작업의 산출 경로를 추적할지"
definition: "CollaborationPattern은 SDI의 7번째 1등 시민 엔티티다. kind ∈ {workflow, graph, swarm, agents-as-tools, direct} 이며, 모든 work entity(plan/requirement/scenario/task/decision/round)는 produced_via_pattern_id를 NOT NULL로 보유한다. direct는 anti-pattern의 명시적 표기다."
relatesTo:
  - { to: concept.collaboration-pattern, type: governs, note: "이 결정이 CollaborationPattern 개념의 지위를 1등 시민으로 격상한다" }
  - { to: domain.collaboration-patterns, type: governs, note: "패턴 도메인의 구조 전체를 이 결정이 정의한다" }
  - { to: domain.scenario-management, type: impacts, note: "시나리오도 어떤 패턴에서 산출됐는지 기록한다" }
  - { to: domain.planning, type: impacts, note: "플랜도 어떤 패턴에서 산출됐는지 기록한다" }
  - { to: domain.decision-negotiation, type: impacts, note: "결정도 패턴 컨텍스트 안에서 이루어진다" }
  - { to: domain.autonomy, type: impacts, note: "패턴 종류별 자율성 기본값(D25)이 이 결정에 의해 성립한다" }
  - { to: domain.agent-coordination, type: governs, note: "에이전트 조율의 형태와 검증이 패턴 엔티티를 기반으로 동작한다" }
supersedes: []
implementedIn:
  - sdi-plugin/crates/core
  - sdi-plugin/plugin/hooks
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI는 AWS 4가지 multi-agent 패턴(Workflow/Graph/Swarm/Agents-as-Tools)을 내장한다(D15). 그런데 이 패턴을 코드 상수나 문서 규약으로만 두면 세 가지 문제가 생긴다. 어떤 작업이 어떤 패턴 아래에서 이루어졌는지 추적 불가, 패턴 무결성을 런타임에 강제할 근거가 없음, 사후 분석("어떤 패턴이 실제로 잘 작동했나")이 불가능하다.

D22를 통해 패턴을 DB 영구 row로 격상하기로 했다. 그러면 D23에서 자연히 따라오는 질문은 "모든 work entity가 어떤 패턴에서 나왔는지를 기록해야 하는가"이고, 답은 "yes"였다.

## 결정 (Decision)

CollaborationPattern을 Plan/Requirement/Decision/Scenario/Round/AutonomyPolicy에 이은 7번째 1등 시민 엔티티로 정한다. 종류는 workflow(고정 순서 step) / graph(DAG 의존, consensus 게이트) / swarm(peer 무계층) / agents-as-tools(한 에이전트가 다른 에이전트를 도구로 호출) / direct(메인 1인극 — anti-pattern 명시 표기) 다섯 가지다.

모든 work entity(plan/requirement/scenario/task/decision/round)는 `produced_via_pattern_id` 컬럼을 NOT NULL로 보유한다. v0.5 이후 신규 생성 entity는 이 컬럼이 반드시 있어야 하고, 없으면 auto로 `direct` 패턴 row를 부여한다. `direct`는 대시보드에서 빨간 배지로 표시되어 1인극이 눈에 띄게 드러나도록 한다.

패턴의 lifecycle은 pending → active → converged | dissensus | aborted 이며, pending → active 전이 시점에 D26 shape validation이 실행된다. 1-step workflow, 단일 인스턴스 swarm 같은 가짜 패턴은 이 검사에서 거부된다.

## 근거와 결과 (Consequences)

패턴을 엔티티로 두면 세 가지가 가능해진다.

첫째, 런타임 게이트: PreToolUse hook이 active pattern의 kind를 조회해 패턴 규칙 위반을 차단할 수 있다. 예를 들어 workflow에서 선행 step이 완료되지 않으면 다음 step을 차단한다.

둘째, 패턴별 자율성 차등: AutonomyPolicy에 pattern_kind scope를 추가하여 workflow=L5, graph=L5, swarm=L4, agents-as-tools=L4, direct=L3 강제로 게임이론적 안전성 차이를 정책으로 반영할 수 있다.

셋째, anti-pattern 가시화: `direct` 패턴이 빨간 배지로 드러남으로써, 사람이 모르는 사이에 solo flow가 쌓이는 것을 설계 수준에서 방지한다.

대가는 모든 작업 생성 경로에 패턴 지정 단계가 추가된다는 마찰이다. 이를 낮추기 위해 패턴 미지정 시 `direct` 자동 부여(명시적 anti-pattern 마커) + pattern-orchestrator 전문 에이전트 타입으로 패턴 선택을 가이드한다.

<!-- provenance: sdi-plugin/docs/PRD.md §2 D22·D23·D24·D25·D26·D27. sdi-plugin/CLAUDE.md D22·D23·D24·D25·D26·D27. sdi-plugin/README.md "CollaborationPattern (D22, v0.5)" 섹션. crates/core/ (v0.5). plugin/hooks/ (D26 gate) -->
