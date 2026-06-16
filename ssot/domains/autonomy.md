---
id: domain.autonomy
kind: Domain
title: 자율성 정책 (Autonomy)
purpose: "에이전트가 합의된 결정을 자동 적용(L5), 통보 후 적용(L4), 항상 확인(L3) 중 어떤 방식으로 처리할지를 스코프(플랜·결정 종류·패턴 종류)별로 설정하고, 빌더가 언제든 회로 차단기로 즉시 L3로 강등할 수 있게 한다."
definition: "AutonomyPolicy 엔티티(SDI 6번째 1등 시민)의 설정·조회·circuit breaker 작동을 담당하는 경계. 정책은 SNAPSHOT-ONLY — 최신 상태만 유효. 스코프는 plan·decision_kind·pattern_kind. 기본값: 신규 개발 플랜 L5, 외부 노출 플랜 L4, architecture·schema·naming-canonical 결정 종류 L4 강제, direct 패턴 L3 강제. circuit breaker는 단일 액션으로 모든 스코프를 즉시 L3로 강등한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.decision-negotiation
    type: relates-to
    note: 합의된 결정에 AutonomyPolicy가 적용되어 자동 실행 여부와 사용자 게이트 위치를 결정한다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: 패턴 종류별 기본 자율 모드가 AutonomyPolicy에 설정된다. 패턴 모드와 플랜 모드 중 더 엄격한 쪽이 우선한다.
  - to: domain.governance-audit
    type: relates-to
    note: 자율 모드 변경, circuit breaker 작동, bypass 사용이 모두 감사 로그에 기록된다.
  - to: concept.autonomy-policy
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.circuit-breaker
    type: relates-to
    note: 빌더가 즉시 모든 자율성을 L3로 강등하는 단일 액션 안전 장치다.
  - to: concept.usage-budget
    type: relates-to
    note: L5 자동 적용의 추가 조건으로 blast_radius_score ≤ l5_threshold(기본 5) 검사가 있다.
governedBy: []
realizedBy: []
impacts:
  - domain.decision-negotiation
  - domain.collaboration-patterns
implementedIn:
  - "sdi-plugin/crates/core/src/autonomy_policy.rs"
  - "sdi-plugin/crates/daemon/src/router/autonomy_policy.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI의 핵심 가치 명제 하나를 기술적으로 보장한다 — "사용자가 PC 앞에 묶이지 않아도 된다." 에이전트가 매 결정마다 확인을 요청하면(default=ask) 빌더는 항상 대기해야 한다. SDI는 반대 방향을 선택했다 — "정책대로 진행하고, 사용자는 개입이 필요한 지점만 정한다(D16)." 이 도메인이 그 정책을 스코프별로 정밀하게 설정하는 장치다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **세 가지 자율 수준**:
  - **L5**: 즉시 자동 적용 + 사후 증거 기록. 빌더 확인 없음.
  - **L4**: 빌더에게 통보 후 타임아웃 내 이의 없으면 자동 적용. 외부 노출 플랜의 기본값.
  - **L3**: 항상 빌더 확인 필요. circuit breaker 작동 시 전 스코프가 즉시 L3로 강등.

- **스코프별 설정**: 같은 빌더라도 플랜마다, 결정 종류마다, 패턴 종류마다 다른 자율 수준을 가질 수 있다. 충돌 시 더 엄격한 쪽이 우선한다.

- **강제 L4 종류(D17)**: architecture·schema·naming-canonical 결정은 자율 모드와 무관하게 항상 L4 이하. 되돌리기 비용이 크기 때문이다.

- **L5 추가 조건(D28)**: L5가 활성화되려면 합의 결정에 되돌림 계획(reversal_plan)이 있고 blast_radius_score가 l5_threshold 이하여야 한다. 되돌릴 수 없는 결정은 L5 자동 적용이 안 된다.

- **circuit breaker(D18)**: 빌더의 단일 액션으로 모든 스코프의 자율 모드가 즉시 L3로 강등. 이미 진행 중인 결정은 다음 게이트에서 적용. 빌더가 "뭔가 이상한데"라는 직관이 들면 즉시 통제권을 되찾는 안전 장치.

- **SNAPSHOT-ONLY**: 정책은 상태 전이 없이 최신 값으로만 유지된다. circuit breaker 액션이 정책 값을 즉시 변경한다.

포함되지 않는 것: 결정 내용 자체(domain.decision-negotiation), 패턴 shape 검증(domain.collaboration-patterns).

## 기능

- 플랜·결정 종류·패턴 종류 스코프별 자율 수준을 설정한다.
- 합의 결정 적용 시 스코프별 정책을 조회해 자동 적용, 통보, 또는 확인 요청을 결정한다.
- circuit breaker 액션으로 전 스코프를 즉시 L3로 강등한다.
- L5 적용 전 되돌림 계획 존재와 blast_radius_score를 검증한다.
- pattern_depth_cap과 plan_single_session_lock 설정을 관리한다.

## 시스템 흐름

빌더가 플랜을 만들 때 기본 자율 모드가 자동 설정된다(신규 개발 → L5, 외부 노출 → L4). 에이전트들이 협상으로 합의에 이르면 데몬이 스코프별 AutonomyPolicy를 조회한다. L5면 즉시 적용, L4면 빌더에게 통보 후 타임아웃, L3면 확인 요청. 빌더가 이상함을 느끼면 대시보드의 circuit breaker를 작동시켜 즉시 모든 스코프를 L3로 강등한다.

## 다른 도메인과의 관계

자율성 정책은 결정 협상 결과가 실제 코드 변경으로 이어지는 마지막 게이트다. 협상이 "합의됐는가"를 판단하면, 이 도메인이 "그 합의를 얼마나 믿고 자동 실행할 것인가"를 결정한다.

## 미확정 (OPEN)
- [ ] OPEN: L4 타임아웃 기본값(시간)이 PRD에 명시적으로 정의되어 있지 않음 — 구현 코드에서 확인 필요.
