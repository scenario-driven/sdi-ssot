---
id: domain.requirements
kind: Domain
title: 요구사항 (Requirements)
purpose: "빌더가 자연어로 제시한 요구사항을 스냅샷 형태로 플랜에 귀속시켜, LLM 에이전트가 태스크 분해 시 컨텍스트로 참조하게 한다. 변경 이력은 본문에 남기지 않고 Decision 엔티티에 위임한다."
definition: "Requirement 엔티티의 생성·조회·최신 스냅샷 갱신을 담당하는 경계. 요구사항은 SNAPSHOT-ONLY다 — 가장 최신 스냅샷만 유효하고, 이전 내용은 본문에서 사라지며, 변경 이력은 append-only Decision에 분리 보존된다. 상태 전이 없음."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: 요구사항이 시나리오의 배경 컨텍스트를 제공한다. 태스크 분해 시 시나리오와 요구사항을 함께 참조한다.
  - to: domain.task-decomposition
    type: relates-to
    note: scenario-decomposer가 시나리오에 더해 요구사항을 읽어 태스크를 도출한다.
  - to: domain.planning
    type: relates-to
    note: 요구사항은 플랜에 귀속된다. 플랜 컨텍스트 안에서만 유효하다.
  - to: concept.requirement
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
governedBy: []
realizedBy: []
impacts:
  - domain.task-decomposition
implementedIn:
  - "sdi-plugin/crates/core/src/requirement.rs"
  - "sdi-plugin/crates/daemon/src/router/requirement.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 빌더가 자유 자연어로 표현한 요구("이런 기능이 필요하다", "이 제약을 지켜야 한다")를 LLM이 읽을 수 있는 구조화된 형태로 플랜에 연결한다. 요구사항은 시나리오처럼 GWT 형식을 강제하지 않는다 — 자연어 자유 서술이 허용된다. 대신 항상 최신 스냅샷만 살아있다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **SNAPSHOT-ONLY 갱신**: 요구사항은 상태 전이가 없다. 내용이 바뀌면 그냥 덮어쓴다. "이전에는 A였는데 B로 바뀌었다"는 본문에 남기지 않는다. 변경 이력이 필요하면 Decision 엔티티(append-only ADR)에 분리한다(D12).

- **플랜 귀속**: 요구사항은 반드시 하나의 플랜에 귀속된다. 플랜 맥락 바깥의 독립 요구사항은 없다.

- **태스크 분해의 입력**: LLM이 태스크를 분해할 때 시나리오만이 아니라 요구사항도 함께 읽는다. "이 시나리오가 왜 필요한지"의 배경이 요구사항에 담겨 있기 때문이다.

포함되지 않는 것: 시나리오의 GWT 정형화(domain.scenario-management), 태스크 생성(domain.task-decomposition), 결정 이력(domain.decision-negotiation).

## 기능

- 자연어 요구사항을 플랜에 귀속시켜 저장한다.
- 요구사항 내용을 최신 스냅샷으로 in-place 갱신한다.
- 플랜 내 요구사항 목록을 조회한다.
- 태스크 분해 시 시나리오와 함께 LLM이 참조할 컨텍스트를 제공한다.

## 시스템 흐름

빌더가 플랜에 요구사항을 추가하면 데몬이 저장한다. 요구사항이 변경되면 기존 내용을 덮어써 최신 스냅샷으로 유지한다. scenario-decomposer 서브에이전트가 `sdi requirement view <REQ-ID>`로 내용을 읽어 태스크 제목과 범위를 도출한다. 요구사항 변경의 이력은 이 도메인이 아니라 Decision 엔티티에 기록하는 것이 설계 원칙이다.

## 다른 도메인과의 관계

요구사항은 "무엇을 만들 것인가"의 언어적 근거다. 시나리오가 "어떻게 동작해야 하는가"의 정형 표현이라면, 요구사항은 그 시나리오의 배경과 이유다. 두 도메인은 태스크 분해의 입력으로 함께 사용되어 서로를 보완한다.

## 미확정 (OPEN)
- [ ] OPEN: 요구사항 변경 시 자동으로 Decision을 생성하는 흐름이 현재 구현에 포함되어 있는지 확인 필요.
