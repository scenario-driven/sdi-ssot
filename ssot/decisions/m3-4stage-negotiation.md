---
id: decision.m3-4stage-negotiation
kind: Decision
title: 에이전트 간 협상은 M3 4단계 구조(제안→비평→합의/이견)를 따른다
purpose: "여러 LLM 에이전트가 설계·결정에 대해 어떻게 의견을 조율할지 — 자유 대화로 갈지, 구조화된 협상 프로토콜로 갈지"
definition: "에이전트 간 협상(M3 Negotiation)은 제안(Proposal) → 비평(Critique) → 합의(Consensus) 또는 이견(Dissensus) 4단계로 구조화된다. 단일 에이전트 제안은 L3(항상 사용자 확인)을 넘을 수 없고, multi-agent consensus만 L4/L5 자동 적용을 unlock한다."
relatesTo:
  - { to: concept.consensus, type: governs, note: "이 결정이 Consensus 개념의 도달 조건과 형식을 정의한다" }
  - { to: concept.decision, type: governs, note: "Decision.kind(proposal/critique/consensus/dissensus)의 4값이 이 협상 단계를 반영한다" }
  - { to: domain.decision-negotiation, type: governs, note: "협상 도메인의 구조 전체가 이 결정 위에 서 있다" }
  - { to: domain.collaboration-patterns, type: impacts, note: "Graph 패턴의 consensus 게이트가 이 협상 구조를 따른다" }
  - { to: domain.autonomy, type: impacts, note: "L3/L4/L5 자율성 잠금 해제 조건이 협상 결과에 의존한다" }
  - { to: concept.collaboration-pattern, type: relates-to, note: "CollaborationPattern의 Graph kind가 이 협상 구조의 컨테이너다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI는 multi-agent orchestration이 본체다(D13). 여러 LLM 에이전트가 설계 결정에 참여할 때 "누가 결정권을 갖는가"와 "어떤 과정을 거쳐야 그 결정을 신뢰할 수 있는가"가 핵심 문제다.

자유 대화 방식에서는 에이전트 하나가 제안하면 다른 에이전트들이 임의 순서로 응답하고, 어느 시점에 "합의됐다"고 볼 수 있는지 기준이 없다. 이 불명확성은 LLM이 가짜 합의(같은 stance의 에이전트 2개가 형식적으로 동의)를 만들어 자율성을 잘못 unlock하는 구멍을 낳는다.

## 결정 (Decision)

에이전트 간 협상을 Decision entity의 kind 필드로 표현하는 4단계 구조로 정형화한다.

1. **Proposal**: 한 에이전트가 결정 제안을 작성한다. Decision.kind = proposal.
2. **Critique**: 다른 에이전트가 제안에 대한 비평을 작성한다. Decision.kind = critique. Graph 패턴에서는 (AgentSpec.name, AgentSpec.stance) tuple이 ≥ 2개 distinct해야 한다(sybil 차단 — 같은 impl-coder 인스턴스 2개는 독립 판단을 보장하지 않는다).
3. **Consensus**: 비평들을 검토하고 최종 합의안을 작성한다. Decision.kind = consensus. 이 단계만 L4/L5 자율 적용을 unlock할 수 있다.
4. **Dissensus**: 합의에 이르지 못할 경우 결정된다. Decision.kind = dissensus. autonomy mode 무관, 즉시 사람 escalation.

단일 에이전트 결정은 kind=proposal/critique를 거쳐도 L3(항상 사용자 확인)이 최대치다. `/decide` 슬래시 명령과 `/consensus` 스킬이 이 흐름을 가이드한다.

## 근거와 결과 (Consequences)

4단계 구조로 정형화하면 두 가지가 동시에 성립한다.

첫째, 합의의 신뢰도를 구조적으로 검증할 수 있다. (name, stance) tuple distinctness 요건이 "진짜 다른 관점의 에이전트가 참여했는가"를 측정 가능한 기준으로 제공한다. 형식적 합의(sybil)를 D26 integrity gate가 pending 단계에서 차단한다.

둘째, dissensus가 명시적 상태가 된다. 합의 실패가 "어쩌다 대화가 끝남"이 아니라 Decision entity에 kind=dissensus로 기록되어, 사람이 어떤 지점에서 이견이 발생했는지 추적 가능하다.

대가는 구조를 지키기 위한 프로토콜 비용이다. 에이전트는 자유롭게 대화하는 대신, proposal/critique/consensus 순서를 명시적으로 따라야 한다. `/decide` 명령과 `sdi:decision-resolver` 에이전트 타입이 이 비용을 낮추도록 설계되었다.

<!-- provenance: sdi-plugin/docs/PRD.md §2 D20 "Consensus/Dissensus가 autonomy gate의 단위". sdi-plugin/README.md "Seven first-class entities — Decision" 섹션 "kind ∈ {proposal, critique, consensus, dissensus}". sdi-plugin/CLAUDE.md D20·D26. sdi-plugin/docs/PRD.md §3 Decision.kind -->
