---
id: component.agent-decision-resolver
kind: SystemComponent
title: decision-resolver 서브에이전트
purpose: SDI의 M3 4단계 협상(제안→비평→합의|불합의, D20)을 주도하는 전문 에이전트다. 사소하지 않은 선택이 코드에 반영되기 전에 내구성 있는 근거를 갖춘 합의를 만들어내거나, 합의 불가 시 명시적으로 불합의를 기록하고 사용자에게 에스컬레이션한다.
realizedBy:
  - domain.decision-negotiation
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/decision-resolver.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: component.agent-schema-architect
    type: leads-to
    note: architecture·schema·naming-canonical 결정은 schema-architect를 비평자로 필수 포함한다(D17 forced-L4)
  - to: domain.decision-negotiation
    type: backed-by
  - to: domain.governance-audit
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

decision-resolver는 SDI의 의사결정 협상을 전담하는 에이전트다. 코드 변경에 앞서 비사소(non-trivial) 선택 — 아키텍처, 스키마, 네이밍 방향, 알고리즘 선택 등 — 에 근거 있는 합의가 필요할 때 이 에이전트를 스폰한다.

4단계 협상 프로토콜은 엄격하다. 첫째, decision-resolver가 `proposal` 행을 작성하고 데몬에 기록한다. 둘째, 비평자가 `critique` 행을 작성한다(비평 없는 합의는 데몬이 HTTP 400으로 거부한다). 셋째, 합의 가능하면 `consensus`를 내보내고, 합의 불가능하면 `dissensus`를 기록한다. 넷째, `dissensus`는 사용자에게 에스컬레이션하며 에이전트가 침묵으로 재시도하지 않는다.

architecture·schema·naming-canonical 결정 종류(D17)는 전역 자율 수준이 L5이더라도 L4 강제 게이트가 걸린다. 이 경우 사용자 프롬프트를 거쳐야 합의가 적용된다.

`reversal_plan`과 `blast_radius_score`가 유효해야 L5 자율 수준이 잠금 해제된다(D28). 이 조건을 채우지 못하면 자율 적용이 차단되므로, decision-resolver는 제안 작성 시 이 두 필드를 반드시 기입한다.

## 경계와 의존

decision-resolver는 Bash·Read 도구만 갖는다. 코드 파일을 수정하지 않으며, `sdi decision create` / `sdi decision critique` / `sdi decision consensus|dissensus` CLI 명령을 통해 데몬에 협상 행을 기록한다.

## 통신 패턴

메인 세션 오케스트레이터가 decision-resolver를 스폰하고, 협상 완료 후 결과(합의 decision id 또는 불합의 에스컬레이션)를 메인 세션에 반환한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/decision-resolver.md`)로 구성된다. 4단계 협상 프로토콜과 불변식이 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: `dissensus` 에스컬레이션 후 사용자가 개입해 강제 합의를 내릴 수 있는 경로가 CLI·대시보드 어디에 있는지 확인 필요.
