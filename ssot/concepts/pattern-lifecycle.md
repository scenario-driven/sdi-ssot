---
id: concept.pattern-lifecycle
kind: Concept
title: PatternLifecycle(패턴 생애주기)
definition: "CollaborationPattern이 거치는 상태 기계. pending(초안) → active(실행 중) → converged(합의 수렴) | dissensus(불합의) | aborted(중단) 흐름. pending→active 전환 시 D26 형태 게이트가 실행된다. converged/dissensus/aborted는 terminal 상태다."
relatesTo:
  - { to: concept.collaboration-pattern, type: belongs-to, note: "PatternLifecycle은 CollaborationPattern의 상태 기계다." }
  - { to: concept.convergence, type: relates-to, note: "converged는 패턴의 수렴 완결 상태다." }
  - { to: domain.collaboration-patterns, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/pattern.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

PatternLifecycle(패턴 생애주기)은 협업 패턴이 거치는 상태 전이 흐름이다.

- **pending**: 패턴이 생성되었으나 아직 활성화되지 않음. 이 단계에서 패턴의 매니페스트(steps/reviewers/fan_out/peer_registration)를 구성한다.
- **active**: 패턴이 활성화됨. 에이전트들이 이 패턴 아래서 작업을 수행 중. `pending → active` 전환 시 D26 형태 게이트가 실행된다.
- **converged**: 합의 수렴 완결. 에이전트들이 목표에 도달했다.
- **dissensus**: 합의 불발. 에이전트들 간 이견이 해소되지 않아 인간 게이트로 에스컬레이션.
- **aborted**: 중단. 기술적 실패, 인간 개입 등 다양한 이유로 패턴이 중단됨.

`converged`, `dissensus`, `aborted`는 terminal 상태다. 이 상태에 도달한 패턴은 되돌릴 수 없으며, 새 패턴을 만들어야 한다. terminal 전환 시 `decided_at` 타임스탬프가 기록된다.

pending→active 전환 시의 형태 게이트(D26):

| 패턴 종류 | 최소 요건 |
|-----------|-----------|
| workflow | steps ≥ 2 |
| graph | distinct (name, stance) 쌍 ≥ 2 |
| swarm | fan_out ≥ 2 |
| agents-as-tools | peer 등록 ≥ 1 |
| direct | 항상 통과 (단 L3 고정) |

## 엔티티 (DB)

collaboration_patterns 테이블의 `lifecycle` 컬럼에 저장된다. terminal 전환 시 `decided_at` 컬럼이 채워진다.

## API 표면

패턴 lifecycle 전환은 daemon의 패턴 API를 통해 이루어진다. pattern-critic specialist가 pending→active 게이트에서 형태를 검토한다.

## 불변식

- **D26 형태 게이트**: pending→active 시 각 종류별 최소 요건을 충족해야 한다.
- **terminal 비가역**: converged/dissensus/aborted는 이전 상태로 돌아갈 수 없다.

## 구현 위치 (provenance)

`PatternLifecycle` 열거형과 `is_terminal()` 메서드는 `sdi-plugin/crates/core/src/pattern.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: aborted 상태로 전환되는 구체적 조건과 API 경로 확인 필요.
