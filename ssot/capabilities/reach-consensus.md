---
id: capability.reach-consensus
kind: Capability
title: 다중 에이전트 합의(M3 4단계 협상)
purpose: "결정을 둘러싼 제안·비판·합의·이견의 4단계 협상을 여러 에이전트가 수행해 단일 에이전트가 혼자 결론 내리지 않도록 한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/commands/consensus.md
  - sdi-plugin/plugin/commands/consensus.md
relatesTo:
  - { to: concept.consensus, type: mutates, note: "proposal → critique → consensus 또는 dissensus 의 단계 전환을 관리한다" }
  - { to: concept.decision, type: mutates, note: "consensus 결정이 에미트되면 해당 결정이 accepted 상태가 된다" }
  - { to: concept.collaboration-pattern, type: reads, note: "graph 패턴(D26)이 합의 협상의 구조를 정의한다; distinct (name, stance) ≥ 2 강제" }
  - { to: concept.autonomy-policy, type: reads, note: "합의 도달이 L4/L5 자동 적용 게이트를 열거나 L3에서 사람 확인을 요청한다" }
  - { to: concept.handoff, type: relates-to, note: "proposal 단계에 critic 에이전트가 없으면 handoff 노트로 critique 를 요청한다" }
impacts:
  - concept.consensus
  - concept.decision
  - concept.autonomy-policy
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

중요한 결정—특히 아키텍처, 스키마, 네이밍 표준 같은 영향 범위가 큰 결정—은 단 하나의 에이전트가 혼자 내리지 않는다. 제안자(proposer)가 결정안을 만들면, 다른 입장을 가진 비판자(critic)가 반박하고, 두 입력을 종합해 합의(consensus)에 이른다. 합의에 실패하면 이견(dissensus)으로 기록하고 사람에게 에스컬레이션한다.

다중 에이전트 합의가 L4/L5 자율성을 여는 열쇠다(D20). 단일 에이전트만 있으면 L3 이상으로 올라갈 수 없다. graph 패턴에서 같은 이름과 같은 입장을 가진 에이전트가 여럿 등록되는 시빌(sybil) 시도는 데몬이 차단한다.

## 행위

- `sdi consensus status <PLAN-ID>` 로 플랜의 모든 제안과 단계 현황을 조회한다.
- `sdi consensus status <PLAN-ID> --proposal-id <DEC-ID>` 로 특정 제안을 드릴다운한다.
- proposal 단계에 critic 없으면 handoff 노트로 `schema-architect` 등 해당 전문가를 호출한다.
- dissensus 상태에서는 사람의 판단 없이 자율 적용하지 않는다.
- graph 패턴 등록 시 `(reviewer.name, reviewer.stance)` 튜플이 ≥2 개 유일해야 활성화된다.

## 시스템 흐름

proposal 결정 생성(`/decide create`) → critique 결정 생성(반대 에이전트가 같은 `proposal_id` 참조) → 데몬이 critique 없는 consensus 거부 → reconcile 후 consensus 또는 dissensus 에미트 → consensus 이면 자율성 모드에 따라 자동 적용(L5) 또는 사람 확인(L3/L4) → dissensus 이면 `escalated_at` 자동 기록 후 사람 에스컬레이션.

## 어디에 구현되어 있나

`/consensus` 슬래시 명령(`plugin/commands/consensus.md`)이 단계 조회 절차를 정의한다. `consensus` 스킬(`plugin/commands/consensus.md`)이 협상 절차를 정의한다. 데몬 HTTP API가 단계 순서(`proposal → critique → consensus/dissensus`)를 강제하고, critique 없는 consensus를 HTTP 400으로 거부한다.

## 미확정 (OPEN)

- [ ] OPEN: dissensus 에스컬레이션 후 사람이 개입해 강제 결정을 내리는 명시적 API 경로가 있는지 확인 필요.
