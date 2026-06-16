---
id: concept.scenario-id
kind: Concept
title: ScenarioId / short_code(시나리오 식별자)
definition: "시나리오를 식별하는 두 가지 식별자. 시스템이 자동 생성하는 ULID 기반 고유 ID(SCN-...)와 사람이 읽는 플랜 내 순차 번호(short_code, 예: SC-1). short_code는 같은 플랜 내에서 유일하며, 서로 다른 플랜은 같은 short_code를 가질 수 있다."
relatesTo:
  - { to: concept.scenario, type: belongs-to, note: "ScenarioId는 Scenario 엔티티의 식별자다." }
  - { to: concept.ticket-number, type: relates-to, note: "short_code는 티켓 번호와 유사하게 사람이 참조하는 짧은 코드다." }
  - { to: domain.scenario-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/ids.rs
  - sdi-plugin/crates/db/src/migrations/010_short_code_scope.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

시나리오는 두 가지 식별자를 갖는다.

1. **시스템 ID**: `SCN-<ULID>` 형태의 전역 유일 식별자. 시스템이 자동 생성하며 변경되지 않는다. 데이터베이스 외래 키와 API에서 기본 참조 식별자로 사용된다.

2. **short_code**: 플랜 내에서 사람이 읽는 짧은 코드(예: SC-1, SC-2). 플랜 안에서만 유일하며, 서로 다른 플랜은 같은 short_code를 가질 수 있다(마이그레이션 010에서 전역 유일 → 플랜 범위 유일로 변경됨). `depends_on` 필드에서 시나리오 간 의존성을 표현할 때 short_code를 사용한다.

시스템 ID의 ULID 부분은 시간 정보를 포함해 생성 순서를 대략 알 수 있다.

## 엔티티 (DB)

시스템 ID는 scenarios 테이블의 primary key이다. short_code는 (plan_id, short_code) 복합 UNIQUE 제약을 갖는다(마이그레이션 010 이후). 이전에는 short_code가 전역 유일이었으나 운영 중 불편함이 드러나 플랜 범위 유일로 변경되었다.

## API 표면

두 식별자 모두 API에서 사용된다. 내부 참조(FK)는 시스템 ID, 사용자 입력이나 태스크의 depends_on은 short_code를 주로 사용한다.

## 불변식

- **시스템 ID 전역 유일**: SCN-... 형태의 시스템 ID는 전역적으로 유일하다.
- **short_code 플랜 범위 유일**: 같은 플랜 안에서는 short_code가 중복될 수 없다.
- **자기참조 금지**: depends_on에 자기 자신의 short_code를 포함할 수 없다.

## 구현 위치 (provenance)

IdKind::Scenario와 "SCN" 접두사는 `sdi-plugin/crates/core/src/ids.rs`에 있다. short_code 복합 유일 제약은 마이그레이션 010에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: short_code의 형식(숫자만, 영숫자 혼합 등) — 코드에서 형식 강제가 보이지 않아 호출자가 자유롭게 지정 가능한지 확인 필요.
