---
id: concept.autonomy-policy
kind: Concept
title: AutonomyPolicy(자율성 정책)
definition: "SDI의 여섯 번째 1등 시민 엔티티(D14). 에이전트가 결정을 자동으로 적용할 수 있는 수준을 per-scope로 제어한다. L3(인간 확인 필수) / L4(자동 적용+취소 창) / L5(자동 적용+사후 통보) 세 모드가 있으며, 회로 차단기(D18)가 모든 모드를 즉시 L3으로 강등시킬 수 있다."
relatesTo:
  - { to: concept.plan, type: relates-to, note: "scope=plan 정책은 특정 플랜의 모든 결정에 적용된다." }
  - { to: concept.decision, type: relates-to, note: "결정 종류(architecture/schema/naming-canonical)에 따라 강제 L4 규칙이 있다(D17)." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "패턴 종류별 기본 모드가 있으며(D25), 플랜 모드와 패턴 모드 중 더 엄격한 쪽이 유효 모드가 된다." }
  - { to: concept.circuit-breaker, type: relates-to, note: "회로 차단기 발동 시 모든 AutonomyPolicy가 즉시 L3으로 강등된다(D18)." }
  - { to: concept.project, type: belongs-to, note: "AutonomyPolicy는 project_id로 프로젝트에 속한다." }
  - { to: domain.autonomy, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/autonomy_policy.rs
  - sdi-plugin/crates/db/src/migrations/006_v04_multi_agent.sql
  - sdi-plugin/crates/db/src/migrations/007_v05_pattern_enforcement.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

AutonomyPolicy(자율성 정책)는 "에이전트가 얼마나 자율적으로 결정을 적용할 수 있는가"를 제어하는 정책이다. 이 정책은 에이전트 간 통신 채널(M1~M5)을 제어하지 않는다(D19). 오직 consensus 결정이 인간 사용자의 확인을 언제 받는지만 제어한다.

**세 가지 자율성 모드**:

- **L3(ask)**: 모든 consensus 결정을 인간이 개별 확인해야 한다. 가장 엄격한 모드.
- **L4(act-with-review)**: consensus 결정을 자동으로 적용하되, 사용자에게 취소 시간 창(`timeout_ms`)을 준다. 시간 창이 지나면 확정된다.
- **L5(act-and-notify)**: consensus 결정을 즉시 자동 적용하고 사용자에게 사후 통보한다. 세 조건이 모두 충족되어야 한다: (1) 패턴 형태 유효, (2) reversal_plan NOT NULL, (3) blast_radius_score ≤ l5_threshold.

**정책 범위(scope_kind)**:

- **plan**: 특정 플랜에 적용. plan_id 필수.
- **decision_kind**: 특정 결정 종류에 적용(예: "architecture"). decision_kind 필수.
- **pattern_kind**: 특정 패턴 종류에 적용(예: "workflow"). pattern_kind 필수.
- **global**: 전체에 적용. plan_id/decision_kind/pattern_kind 없음.

**기본 모드(D17)**:

- 새 플랜: L5 기본.
- 외부 표면이 있는 플랜(`external_surface=true`): L4 기본.
- architecture/schema/naming-canonical 결정 종류: L4 강제 하한. L5 허용 안 됨.

**패턴별 기본 모드(D25)**:

| 패턴 종류 | 기본 모드 |
|-----------|-----------|
| workflow | L5 |
| graph | L5 |
| swarm | L4 |
| agents-as-tools | L4 |
| direct | L3 고정 |

플랜 레벨 모드와 패턴 레벨 모드가 둘 다 있으면 더 엄격한(낮은) 쪽이 유효 모드가 된다.

## 엔티티 (DB)

AutonomyPolicy 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 프로젝트 참조, 선택적 플랜 참조.
- **범위**: scope_kind, decision_kind(optional), pattern_kind(optional).
- **모드**: L3/L4/L5.
- **L5 임계값(D28)**: l5_threshold(기본 5). blast_radius_score가 이 값 이하일 때만 L5 자동 적용 허용.
- **깊이 상한(D24)**: pattern_depth_cap(기본 3). 패턴 중첩 최대 깊이.
- **잠금**: plan_single_session_lock(기본 false). true면 플랜의 active 시나리오는 동시에 하나의 세션만 claim할 수 있다.
- **외부 표면**: external_surface(기본 false). true면 이 플랜 기본 모드가 L4.
- **타임아웃**: timeout_ms — L4 자동 적용 대기 시간(ms). NULL이면 자동 적용 없음.
- **강제 여부**: forced(기본 false). 아키텍처/스키마 결정 종류 L4 강제 여부.
- **메타**: set_at, set_by, reason.

## API 표면

AutonomyPolicy는 프로젝트 설정 또는 플랜 설정에서 관리된다. SDI 회로 차단기(D18)가 활성화되면 모든 정책이 즉시 L3으로 강등된다(`sdi bypass arm`으로 긴급 우회 가능).

## 불변식

- **scope 일관성**: scope=plan이면 plan_id 필수, scope=decision_kind이면 decision_kind 필수 등 scope_kind에 맞는 필드만 채워야 한다.
- **architecture/schema/naming-canonical L4 하한(D17)**: 이 종류의 결정에 L5 정책을 설정하면 거부된다.
- **D20 단일 에이전트 L3 상한**: 단일 에이전트는 L3이 최대. 다중 에이전트 consensus 없이 L4/L5 달성 불가.

## 구현 위치 (provenance)

AutonomyPolicy 도메인 규칙(scope 검증, 모드 floor 검사, 패턴별 기본 모드, 유효 모드 계산)은 `sdi-plugin/crates/core/src/autonomy_policy.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: plan_single_session_lock이 실제로 다중 세션 claim을 어떻게 차단하는지 — D29 클레임 메커니즘과의 정확한 연동 확인 필요.
