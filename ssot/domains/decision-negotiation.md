---
id: domain.decision-negotiation
kind: Domain
title: 결정 협상 (Decision Negotiation)
purpose: "에이전트들 간의 결정을 append-only ADR 형식으로 기록하고, M3 4단계 협상(제안→비평→합의/이견)을 통해 다중 에이전트 합의를 형성하거나 빌더에게 에스컬레이션한다."
definition: "Decision 엔티티의 생성·append-only 관리·kind 전이(proposal→critique→consensus|dissensus)를 담당하는 경계. 결정은 절대 수정·삭제되지 않는다. 합의(consensus)가 자율 적용의 단위이며, 이견(dissensus)은 항상 빌더에게 에스컬레이션된다. 아키텍처·스키마·명명 규칙 결정은 합의 후에도 L4 사용자 게이트를 강제한다(D17)."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.autonomy
    type: relates-to
    note: AutonomyPolicy가 합의된 결정의 자동 적용 여부를 결정한다. 단일 에이전트 결정은 L3 최대(항상 사람 확인).
  - to: domain.collaboration-patterns
    type: relates-to
    note: Graph 패턴이 결정 협상의 다양성 보장(distinct (name, stance) ≥ 2)을 강제한다.
  - to: domain.governance-audit
    type: relates-to
    note: 모든 결정의 생성·적용·롤백이 감사 로그에 기록된다.
  - to: concept.decision
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.consensus
    type: relates-to
    note: 자율 적용의 조건이 되는 합의다.
  - to: concept.dispatch
    type: relates-to
    note: 합의된 결정을 자율성 정책에 따라 자동 적용하는 행위다.
governedBy: []
realizedBy: []
impacts:
  - domain.autonomy
  - domain.governance-audit
implementedIn:
  - "sdi-plugin/crates/core/src/decision.rs"
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
  - "sdi-plugin/plugin/agents/decision-resolver.md"
  - "sdi-plugin/plugin/agents/schema-architect.md"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 "결정이 어떻게 내려지는가"를 관리한다. SDI에서 결정은 단순한 선택이 아니라 협상의 산물이다. 한 에이전트의 단독 의견은 L3 최대(항상 사람 확인)이고, 둘 이상의 에이전트가 독립적으로 합의에 이른 결정만 L4/L5 자율 적용이 가능하다(D20). 이 도메인이 그 협상 과정과 결과를 append-only 방식으로 기록한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **4단계 협상 흐름(M3)**: `proposal(제안) → critique(비평) → consensus(합의) | dissensus(이견)`. 비평이 한 개도 없는 proposal에는 consensus를 발행할 수 없다. 이 순서는 데몬이 강제한다.

- **종류(kind)**: `proposal` — 최초 제안. `critique` — 비평(반드시 proposal에 연결). `consensus` — 합의(비평 1개 이상 후 발행 가능). `dissensus` — 이견(에스컬레이션 신호).

- **append-only 불변**: 결정은 수정·삭제되지 않는다(D12). 잘못된 결정을 되돌리려면 롤백 결정(kind=consensus, reversal_of=<원본 ID>)을 새로 추가한다.

- **되돌림 계획(D28)**: 모든 결정은 `reversal_plan`(역 마이그레이션 SQL / git revert SHA / fs 스냅샷 참조 / 보상 액션)과 `blast_radius_score`(0~10, 영향 범위 점수)를 갖는다. L5 자동 적용의 추가 조건은 reversal_plan 존재 + blast_radius_score ≤ l5_threshold(기본 5)이다.

- **강제 L4 종류(D17)**: 아키텍처·스키마·명명 규칙 결정은 합의에 도달하더라도 반드시 빌더의 검토를 거친다. 이 세 종류는 자율 모드가 L5여도 L4로 강제 강등된다.

포함되지 않는 것: 자율성 정책 설정(domain.autonomy), 패턴 선택(domain.collaboration-patterns), 롤백 실행(reversal-runner 서브에이전트).

## 기능

- 제안·비평·합의·이견 Decision을 append-only로 추가한다.
- 비평 없는 합의 발행을 거부한다.
- dissensus 발생 시 빌더에게 에스컬레이션 신호를 보낸다.
- 되돌림 계획과 영향 점수를 검증한다.
- 아키텍처·스키마·명명 규칙 종류에 대해 L4 강제를 적용한다.
- 합의 결정에 대해 자율성 정책에 따라 자동 적용 또는 통보한다.

## 시스템 흐름

decision-resolver 서브에이전트가 제안 Decision을 만든다. schema-architect(또는 도메인별 비평 전문가)가 비평 Decision을 추가한다. 비평이 1개 이상 쌓이면 decision-resolver가 합의 또는 이견을 발행한다. 합의라면 자율성 정책에 따라 자동 적용(L5), 통보 후 적용(L4), 또는 사람 확인(L3)을 거친다. 이견이라면 빌더에게 에스컬레이션되고 서브에이전트가 임의 재시도하지 않는다.

## 다른 도메인과의 관계

결정 협상은 자율성(domain.autonomy)과 긴밀하게 연결된다. 자율성 정책이 "어떤 수준의 합의가 자동 적용을 허용하는가"를 정의하고, 이 도메인이 "합의가 실제로 이루어졌는가"를 기록한다. 합의 여부가 자율 적용의 게이트 역할을 한다.

## 미확정 (OPEN)
- [ ] OPEN: dissensus 에스컬레이션 시 빌더에게 실제로 어떤 방식(UI 알림, CLI 메시지, SSE 이벤트)으로 전달되는지 구현 확인 필요.
