---
id: concept.ticket-number
kind: Concept
title: Ticket Number / short_code(티켓 번호)
definition: "사람이 읽는 짧은 식별자 패턴. 플랜(P-1), 시나리오(SC-1), 태스크(T-1), 라운드(R-1) 등 각 엔티티가 플랜 범위에서 고유한 short_code를 갖는다. 프로젝트 key(예: SDI)와 조합해 SDI-1처럼 표현할 수 있다."
relatesTo:
  - { to: concept.scenario-id, type: relates-to, note: "시나리오의 short_code가 ticket-number 패턴의 대표적 사용 사례다." }
  - { to: concept.task, type: relates-to, note: "태스크의 short_code는 플랜 범위에서 유일하며 티켓 번호로 참조된다." }
  - { to: concept.project, type: relates-to, note: "프로젝트 key가 티켓 번호의 접두사가 된다." }
  - { to: domain.project-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/ids.rs
  - sdi-plugin/crates/db/src/migrations/010_short_code_scope.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Ticket Number(티켓 번호)는 엔티티를 사람이 기억하고 참조하기 쉬운 짧은 코드다. SDI의 모든 주요 엔티티는 시스템이 생성한 ULID 기반 고유 ID와 함께 사람이 읽는 short_code를 갖는다.

각 엔티티 종류별 typical short_code 형태:

- **플랜**: P-1, P-2, ...
- **시나리오**: SC-1, SC-2, ...
- **태스크**: T-1, T-2, ...
- **라운드**: R-1, R-2, ...
- **요건**: REQ-1, ...
- **결정**: DEC-1, ...

short_code의 유일성 범위는 마이그레이션 010 이후 **플랜 범위**다. 즉, 같은 플랜 안에서는 중복될 수 없지만 다른 플랜은 같은 short_code를 가질 수 있다. 이 덕분에 여러 플랜에 걸쳐 작업할 때 "SC-1"이 어느 플랜에 있는지 맥락을 통해 구별해야 한다.

프로젝트 key와 조합하면 `SDI-T-1`, `SDI-SC-3`처럼 전역적으로 참조 가능한 형태가 된다. 그러나 이 조합 형태가 DB에 저장되지는 않으며 표시 목적으로 조립된다.

## 엔티티 (DB)

각 엔티티 테이블(plans, scenarios, tasks, rounds, requirements, decisions, collaboration_patterns)에 `short_code TEXT` 컬럼이 있으며, `(plan_id, short_code)` 복합 UNIQUE 제약을 갖는다.

## API 표면

short_code는 API 응답에 항상 포함된다. depends_on 같이 에이전트가 다른 엔티티를 참조할 때 short_code를 사용한다.

## 불변식

- **플랜 범위 유일(마이그레이션 010 이후)**: 같은 플랜 안에서 같은 short_code를 가진 두 엔티티는 허용되지 않는다.

## 구현 위치 (provenance)

short_code의 플랜 범위 UNIQUE 제약은 마이그레이션 010에서 정의된다. IdKind별 시스템 ID 접두사는 `sdi-plugin/crates/core/src/ids.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: short_code 자동 생성 로직(순차 번호 부여) — caller가 직접 지정하는지 daemon이 자동 부여하는지 확인 필요.
