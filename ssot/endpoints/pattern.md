---
id: endpoint.pattern
kind: Endpoint
title: "GET /patterns/:id"
definition: 단일 협업 패턴을 식별자로 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
relatesTo:
  - to: concept.collaboration-pattern
    type: reads
    note: "id 로 지정된 하나의 패턴 전체 필드를 반환한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

패턴 식별자를 경로 파라미터로 받아 해당 협업 패턴의 전체 정보를 반환한다. 패턴 유형별 형상 필드(steps, reviewers, fan_out, peer_registration)를 포함하며, 라이프사이클 전이 이력도 함께 제공한다.

## 요청 / 응답

**GET /patterns/:id**

경로 파라미터:

- `id` — 조회할 패턴의 식별자. 필수.

응답 필드:

- `id`, `plan_id`, `short_code`, `kind`, `applies_to`, `scope_id` — 기본 식별 정보.
- `parent_pattern_id` — 상위 패턴이 있으면 해당 식별자, 없으면 null.
- `depth` — 루트로부터의 중첩 깊이.
- `lifecycle` — 현재 라이프사이클 상태(`pending`, `active`, `converged`, `dissensus`, `aborted`).
- `steps` — `workflow` 유형일 때의 단계 정의. JSON 문자열로 직렬화된 형태.
- `reviewers` — `graph` 유형일 때의 검토자 목록. JSON 문자열로 직렬화된 형태.
- `fan_out` — `swarm` 유형일 때의 병렬 분기 수. JSON 문자열로 직렬화된 형태.
- `peer_registration` — `agents-as-tools` 유형일 때의 피어 등록 설정. JSON 문자열로 직렬화된 형태.
- `decided_at` — 종료 상태(converged/dissensus/aborted)로 전환된 시각. 미전환이면 null.
- `decided_reason` — 종료 전환 이유 텍스트. 미전환이면 null.

패턴이 존재하지 않으면 404 를 반환한다.

## 권한 / 제약

- 읽기 전용 엔드포인트다. 상태 변경이 없다.
- 형상 필드(steps, reviewers, fan_out, peer_registration)는 유형에 따라 하나만 유효하고 나머지는 null 이다.
- `decided_at` 과 `decided_reason` 은 종료 상태에 도달해야만 값이 채워진다.

## provenance

협업 패턴이 D22 에서 7번째 1등급 엔티티로 확정되었으며, 형상 필드의 구성은 `workflow`, `graph`, `swarm`, `agents-as-tools`, `direct` 다섯 가지 유형 정의에서 비롯되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/pattern.rs` 에 위치한다.

## 미확정 (OPEN)

- 형상 필드가 실제로 JSON 문자열로 저장되는지, 아니면 구조화된 컬럼으로 분리되어 있는지 구현 확인이 필요하다.
- `depth` 가 저장 컬럼인지 조회 시 동적으로 계산되는 값인지 명시되지 않았다.
- `decided_reason` 을 설정하는 시점(라이프사이클 전환 요청 본문에서 받는지, 별도 메커니즘인지)이 이 엔드포인트 외부에서 결정된다.
