---
id: concept.collaboration-pattern
kind: Concept
title: CollaborationPattern(협업 패턴)
definition: "SDI의 일곱 번째 1등 시민 엔티티(D22). 어떤 에이전트들이 어떤 방식으로 작업 엔티티(플랜/시나리오/태스크/결정 등)를 생산하는지를 선언한다. Workflow/Graph/Swarm/AgentsAsTools/Direct 다섯 종류가 있으며, Direct는 단독 실행 표시(anti-pattern badge)다."
relatesTo:
  - { to: concept.plan, type: relates-to, note: "패턴은 plan_id로 특정 플랜에 소속된다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "패턴 종류별 기본 autonomy mode가 있으며(D25), 플랜 모드와 패턴 모드 중 더 엄격한 것이 적용된다." }
  - { to: concept.decision, type: relates-to, note: "모든 결정은 produced_via_pattern_id로 어떤 패턴 아래 만들어졌는지 추적된다." }
  - { to: concept.consensus, type: relates-to, note: "D28 L5 unlock 조건 중 하나가 패턴 형태 유효성이다." }
  - { to: concept.pattern-lifecycle, type: relates-to, note: "패턴은 pending → active → converged | dissensus | aborted 생애주기를 따른다." }
  - { to: domain.collaboration-patterns, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/pattern.rs
  - sdi-plugin/crates/db/src/migrations/007_v05_pattern_enforcement.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

CollaborationPattern(협업 패턴)은 "이 작업 엔티티를 어떤 방식으로 여러 에이전트가 함께 만들 것인가"를 선언하는 엔티티다. SDI의 기본 철학(D13)은 단일 에이전트 solo 흐름이 anti-pattern이며, 모든 작업은 다중 에이전트 협업을 1등 시민으로 고려해야 한다는 것이다.

패턴은 어떤 종류의 작업 엔티티(`applies_to`: plan/requirement/scenario/task/decision/round)에 적용되는지, 어떤 협업 방식을 쓰는지를 선언한다.

다섯 가지 패턴 종류:

- **workflow**: 순서가 있는 단계별 실행. 최소 2개의 step이 필요하다(D26). 각 step은 에이전트 이름과 수행 액션으로 구성된다.
- **graph**: 동료 peer review 방식. 최소 2개의 고유 `(name, stance)` 튜플 reviewer가 필요하다(D26 sybil 차단). 같은 이름의 에이전트라도 stance가 다르면 구별된 것으로 본다.
- **swarm**: 병렬 fan-out 실행. 최소 2개의 fan-out 대상이 필요하다(D26).
- **agents-as-tools**: caller가 callee를 도구(tool)로 등록해 사용하는 방식. peer 등록이 최소 1개 필요하다(D26).
- **direct**: 단독 실행(solo flow) 표시. 형태 검사를 통과하나 L3 autonomy mode로 고정된다. 새 작업을 단독으로 만드는 것이 anti-pattern임을 명시적으로 드러내는 "anti-pattern badge"다.

패턴은 재귀적으로 중첩될 수 있다(`parent_pattern_id`). 이 DAG는 순환이 차단되며 깊이 상한(`AutonomyPolicy.pattern_depth_cap`, 기본 3)이 있다(D24).

모든 작업 엔티티(plan/requirement/scenario/task/decision/round)는 생성 시 `produced_via_pattern_id`를 가져야 한다(D23). 명시하지 않으면 daemon이 자동으로 `direct` 패턴을 부여한다.

## 엔티티 (DB)

패턴 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜 참조.
- **식별**: 플랜 내 고유 short_code.
- **종류**: workflow/graph/swarm/agents-as-tools/direct.
- **적용 대상**: applies_to(plan/requirement/scenario/task/decision/round).
- **범위**: scope_id(보통 플랜 ID 또는 특정 작업 엔티티 ID).
- **중첩**: parent_pattern_id(상위 패턴), depth(현재 깊이).
- **생애주기**: pending/active/converged/dissensus/aborted.
- **매니페스트**: steps(workflow), reviewers(graph), fan_out(swarm), peer_registration(agents-as-tools) — 해당 종류에 맞는 필드만 채워진다.
- **결정 메타**: decided_at, decided_reason.
- **타임스탬프**: 생성, 수정 시각.

## API 표면

패턴은 `/sdi pattern` 명령(/pattern 슬래시 커맨드 포함)으로 생성·전환·조회한다. `pending → active` 전환 시 D26 형태 검사가 실행된다.

## 불변식

- **D26 형태 게이트**: `pending → active` 전환 시 각 종류별 최소 요건을 충족해야 한다(workflow: step ≥ 2, graph: distinct (name,stance) ≥ 2, swarm: fan_out ≥ 2, agents-as-tools: peer ≥ 1).
- **D23 출처 필수**: 모든 작업 엔티티는 created_at 시점에 produced_via_pattern_id를 가져야 한다.
- **D24 깊이 상한**: parent_pattern_id로 형성된 DAG 깊이가 pattern_depth_cap을 초과할 수 없다.
- **Direct = L3 고정(D25)**: direct 패턴에는 autonomy mode L3이 강제된다.
- **Fake 패턴 차단(D27)**: 1단계 workflow나 1인스턴스 swarm은 direct의 L3 상한을 우회할 수 없다.

## 구현 위치 (provenance)

패턴 도메인 규칙(`validate_pattern_shape`, 각 enum 정의, reversal_plan 파싱)은 `sdi-plugin/crates/core/src/pattern.rs`에 있다. DB 스키마는 마이그레이션 007에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: DAG 순환 감지가 daemon에서 어떻게 구현되는지 — 코드 주석에 "topological sort"가 언급되나 실제 구현 위치 확인 필요.
- [ ] OPEN: 패턴 `aborted` 종료 조건이 무엇인지 — `converged`와 `dissensus` 이외 `aborted`가 언제 발생하는지 확인 필요.
