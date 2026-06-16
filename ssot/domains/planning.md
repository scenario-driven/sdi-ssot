---
id: domain.planning
kind: Domain
title: 플랜 관리 (Planning)
purpose: "빌더의 개발 의도를 Plan으로 구조화하고, 시나리오가 충분히 갖춰졌을 때 빌더의 명시적 승인으로 active 상태로 전환해 라운드 실행을 허용한다."
definition: "Plan 엔티티의 생성·승인·완료를 담당하는 경계. 플랜은 '승인된 의도'다 — draft 상태에서는 라운드를 시작할 수 없고, 빌더가 명시적으로 승인해 active로 전환해야 한다. 승인 게이트 조건: 시나리오 1개 이상 + 각 시나리오 GWT 모두 유효(D8). 태스크 수는 승인 게이트와 무관하다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: 플랜 승인의 조건이 시나리오 존재 여부다. 확인된 GWT 시나리오가 1개 이상 있어야 승인할 수 있다.
  - to: domain.round-execution
    type: relates-to
    note: 플랜이 active여야 그 플랜의 라운드를 시작할 수 있다.
  - to: domain.autonomy
    type: relates-to
    note: 플랜 단위로 AutonomyPolicy를 설정한다. 플랜 스코프의 자율성 모드가 결정 적용 방식을 결정한다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: 플랜 자체도 produced_via_pattern_id를 갖는다(D23). 패턴 기반으로 플랜이 수립됐는지 추적한다.
  - to: concept.plan
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: domain.project-management
    type: relates-to
    note: 플랜은 반드시 하나의 프로젝트에 귀속된다.
governedBy: []
realizedBy: []
impacts:
  - domain.round-execution
  - domain.autonomy
implementedIn:
  - "sdi-plugin/crates/core/src/plan.rs"
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 빌더의 개발 의도가 구현에 앞서 명시적으로 승인되는 관문이다. 플랜이 없으면 라운드를 시작할 수 없고, 플랜이 승인(active)되지 않으면 태스크를 시작할 수 없다. 이 순서는 SDI가 "계획 없는 실행"을 구조적으로 불가능하게 만드는 장치다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **플랜 생성**: 프로젝트에 귀속된 의도의 컨테이너. 어떤 목표를 달성하기 위한 작업 집합인지를 자연어 제목·설명으로 표현한다.

- **승인 게이트(D8)**: `draft → active` 전환을 위한 두 조건 — (1) 시나리오 최소 1개, (2) 각 시나리오의 Given·When·Then이 모두 비어 있지 않음. 태스크 수는 무관하다. 태스크는 라운드 시작 후 LLM이 런타임에 생성하므로, 승인 시점에 존재하지 않는 것이 정상이다.

- **상태 전이**: `draft → active → completed`. active 상태의 플랜이 있어야 그 플랜 소속 라운드를 시작할 수 있다.

- **자율성 정책 스코프**: 각 플랜은 자신만의 AutonomyPolicy를 가질 수 있다. 신규 개발 플랜은 기본 L5(즉시 자동 적용), 외부 노출 플랜(배포·API 공개)은 기본 L4(통보 후 자동 적용)로 시작한다(D17).

포함되지 않는 것: 라운드 실행(domain.round-execution), 시나리오 상태 관리(domain.scenario-management), 요구사항 내용(domain.requirements).

## 기능

- 플랜을 생성하고 프로젝트에 연결한다.
- 승인 게이트 조건을 검증하고 active로 전환한다.
- 플랜 스코프의 AutonomyPolicy를 조회하고 결정에 적용한다.
- 플랜의 모든 라운드가 완료되면 completed로 전환한다.

## 시스템 흐름

빌더가 플랜을 만들고 시나리오를 추가해 confirmed 상태로 만든다. 충분히 쌓이면 `/plan approve` 또는 `sdi plan approve <PLAN-ID>`를 실행한다. 데몬이 승인 게이트 조건(시나리오 ≥1, GWT 모두 유효)을 검사하고 통과하면 active로 전환한다. 이후 코딩 에이전트가 이 플랜에 라운드를 생성하고 태스크를 분해할 수 있게 된다.

## 다른 도메인과의 관계

플랜은 SDI 전체 흐름의 출발점이다. 시나리오가 플랜에 귀속되고, 라운드가 플랜 아래에서 실행되고, 자율성 정책이 플랜 단위로 설정된다. 모든 work entity가 어느 플랜에서 나왔는지 추적된다.

## 미확정 (OPEN)
- [ ] OPEN: 플랜 완료(completed) 전환 조건이 "모든 라운드 완료"인지 "빌더 명시 완료"인지 PRD에서 명확히 정의되지 않은 부분 확인 필요.
