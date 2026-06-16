---
id: component.core
kind: SystemComponent
title: sdi-core (Rust 도메인 모델 크레이트)
purpose: SDI의 7개 퍼스트클래스 엔티티 — Plan, Requirement, Decision, Scenario(GWT), Round, AutonomyPolicy, CollaborationPattern — 의 타입 정의·유효성 규칙·상태 전이 규칙을 순수하게 보유하는 크레이트다. 외부 I/O 없이 도메인 언어를 코드로 표현하는 유일한 장소이며, 나머지 모든 크레이트(cli, daemon, db, mcp)는 이 크레이트를 공통 언어로 삼는다.
realizedBy:
  - domain.scenario-management
  - domain.requirements
  - domain.task-decomposition
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - domain.autonomy
  - domain.collaboration-patterns
implementedIn:
  - sdi-plugin/crates/core
dependsOn: []
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.cli
  - component.daemon
  - component.db
  - component.mcp
relatesTo:
  - to: domain.scenario-management
    type: backed-by
  - to: domain.planning
    type: backed-by
  - to: domain.decision-negotiation
    type: backed-by
  - to: domain.autonomy
    type: backed-by
  - to: domain.collaboration-patterns
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

sdi-core는 SDI 시스템 전체의 도메인 언어를 보유하는 라이브러리 크레이트다. 이 크레이트가 정의하는 것은 크게 두 종류다.

첫째, 7개의 퍼스트클래스 엔티티 타입이다. **Plan**은 시나리오와 요구사항이 승인 게이트(시나리오 1개 이상, 모든 GWT 유효)를 통과해야 활성화되는 작업 묶음이다. **Requirement**는 스냅샷(불변) 성격의 요구사항으로, 한 번 기록된 내용은 교체하지 않고 새 버전을 append한다. **Decision**은 제안→비평→합의|불합의의 4단계 협상(M3, D20)을 거치는 의사결정 레코드로, 되돌리기 계획(reversal_plan)과 영향 범위 점수(blast_radius_score)를 필수로 갖는다. **Scenario**는 Given/When/Then 세 절 모두 비어 있으면 안 되는(D5 GWT-strict) 행동 명세다. **Round**는 R1(신규 개발)과 R2+(회귀 검증) 사이클로, 기본이 strict-regression 모드(D6)다. **AutonomyPolicy**는 플랜·결정 종류·패턴 종류별로 자율 수준(L3~L5)을 기록하는 정책 엔티티다. **CollaborationPattern**은 workflow·graph·swarm·agents-as-tools·direct 다섯 가지 종류를 갖는 협업 형상 엔티티다.

둘째, 각 엔티티의 불변식과 상태 전이 규칙이다. 예를 들어 Plan은 시나리오 0개인 상태에서 승인될 수 없고, Decision의 architecture·schema·naming-canonical 종류는 무조건 L4 강제 게이트(D17)를 적용하며, CollaborationPattern의 `direct`는 L3 상한을 초과할 수 없다. 이 규칙들이 코드에 박혀 있어 다른 크레이트가 이를 재구현하지 않아도 된다.

## 경계와 의존

sdi-core는 외부 의존이 없는 순수 도메인 크레이트다. I/O·HTTP·DB·비동기 런타임을 포함하지 않는다. 직렬화(serde)와 고유 ID 생성(ULID), 날짜 처리(chrono), 슬러그 생성(slug) 정도만 의존한다. 다른 크레이트들이 sdi-core를 의존하며, sdi-core는 어떤 크레이트도 의존하지 않는다.

## 통신 패턴

크레이트 경계를 넘는 런타임 통신이 없다. 컴파일 타임에 타입을 공유하는 방식으로 도메인 계약을 전파한다. 다른 크레이트가 sdi-core의 타입을 직접 사용해 직렬화·DB 매핑·HTTP 응답 변환을 수행한다.

## 하위 서브패키지 (책임 단위)

- **엔티티 타입군**: Plan, Requirement, Decision, Scenario, Round, AutonomyPolicy, CollaborationPattern의 구조체·열거형 정의와 파생 직렬화.
- **유효성 규칙**: GWT strict 검사(D5), Plan 승인 게이트 조건(D8), Decision 필드 필수 규칙(D28).
- **상태 전이 규칙**: Plan(draft→active→completed), Round(R1/R2+), CollaborationPattern(pending→active→converged|dissensus|aborted) 전이 가드.
- **AutonomyPolicy 로직**: 범위별 자율 수준 결정, L4 강제 조건(D17), L5 잠금 해제 조건(D28, blast_radius_score ≤ threshold).

## 미확정 (OPEN)

- [ ] OPEN: Task는 런타임 아티팩트(D3)라서 LLM이 분해하는 개념인데, sdi-core에 Task 타입이 포함되는지 아니면 daemon 레이어에서만 다루는지 확인 필요.
