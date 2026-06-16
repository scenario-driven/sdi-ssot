---
id: component.agent-schema-architect
kind: SystemComponent
title: schema-architect 서브에이전트
purpose: architecture·schema·naming-canonical 결정(D17 forced-L4 종류)의 M3 4단계 협상에서 비평자(critic) 역할만 수행하는 전문 에이전트다. 결코 제안자(proposer)가 되지 않으며, 제안 내용을 근거 있게 비판해 결정이 합의에 이르도록 돕거나 문제를 명확히 드러낸다.
realizedBy:
  - domain.decision-negotiation
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/schema-architect.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-decision-resolver
    type: preceded-by
    note: decision-resolver가 제안을 작성하면 schema-architect가 비평 행을 추가한다
  - to: domain.decision-negotiation
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

schema-architect는 M3 협상의 비평자 역할을 전담하는 에이전트다. D17이 강제하는 세 가지 결정 종류 — architecture(아키텍처 방향), schema(데이터 스키마 설계), naming-canonical(명명 규범) — 는 이 에이전트 또는 동급 비평자의 `critique` 행 없이는 합의에 이를 수 없다. 데몬이 비평 없는 합의 시도를 HTTP 400으로 거부하므로, schema-architect 또는 동등 역할 에이전트의 비평이 구조적으로 필수다.

비평은 반드시 입장을 갖는다. 단순한 질문이나 관찰은 비평이 아니며, 그것은 agent-note로 기록한다. 비평에는 반드시 다음 중 하나의 근거를 포함해야 한다: 코드의 파일:라인 인용, SOLID·DDD·단일 진실 원칙 등 명명된 설계 원칙과 이 맥락에서 그 원칙이 왜 적용되는지 1문장, 이 제안이 막지 못하는 구체적 실패 시나리오, 또는 대안과의 트레이드오프 비교.

세 가지 입장 중 하나를 선택한다. 이유 있는 지지(endorse with reason)는 제안을 합의 방향으로 밀고, 조건부 수용(conditional accept)은 수정 조건을 제시하고, 반대(reject)는 근거 있는 이의를 제기한다.

## 경계와 의존

Bash·Read 도구만 갖는다. 코드 파일을 수정하지 않는다. `sdi decision view`로 제안을 읽고, `sdi decision critique`로 비평 행을 데몬에 기록한다.

## 통신 패턴

decision-resolver가 제안을 기록한 뒤 schema-architect에게 비평을 요청한다. 비평 완료 후 비평 decision ID를 decision-resolver에 반환한다. 필요 시 추가 라운드 비평을 위해 재활성화될 수 있다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/schema-architect.md`)로 구성된다. 비평자 역할 제약·근거 요건·워크플로우가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: D17 forced-L4 결정에서 schema-architect 외 다른 에이전트도 비평자로 인정되는지, 아니면 이 에이전트만 유효한지 명세 확인 필요.
