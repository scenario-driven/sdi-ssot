---
id: endpoint.patterns
kind: Endpoint
title: "POST /patterns, GET /patterns"
definition: 협업 패턴을 생성하거나 플랜에 속한 패턴 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
relatesTo:
  - to: concept.collaboration-pattern
    type: mutates
    note: "POST 는 새로운 협업 패턴을 생성하고, GET 은 목록을 반환한다"
  - to: concept.plan
    type: reads
    note: "plan_id 를 기준으로 패턴 생성·조회가 범위가 한정된다"
  - to: concept.autonomy-policy
    type: reads
    note: "D24: 패턴 중첩 깊이(parent depth)는 AutonomyPolicy 의 pattern_depth_cap(기본값 3)을 기준으로 검증한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

협업 패턴(CollaborationPattern)은 SDI 의 일곱 번째 1등급 엔티티(D22)다. 이 엔드포인트는 플랜 안에서 에이전트들이 어떤 방식으로 협력할지를 정의하는 패턴을 새로 만들거나, 특정 플랜에 등록된 패턴 전체를 나열한다.

## 요청 / 응답

**POST /patterns — 패턴 생성**

요청 본문에 포함해야 하는 필드:

- `plan_id` — 패턴이 속할 플랜 식별자. 필수.
- `short_code` — 패턴을 짧게 지칭하는 코드. 필수.
- `kind` — 패턴 유형. `workflow`, `graph`, `swarm`, `agents-as-tools`, `direct` 중 하나. 필수.
- `applies_to` — 패턴이 적용되는 대상 엔티티 종류. `plan`, `requirement`, `scenario`, `task`, `decision`, `round` 중 하나. 필수.
- `scope_id` — `applies_to` 로 지정한 엔티티의 실제 식별자. 필수.
- `parent_pattern_id` — 이 패턴의 상위 패턴 식별자. 선택. 중첩 패턴을 구성할 때 사용한다.
- `steps` — `workflow` 유형일 때 사용하는 단계 정의. 선택.
- `reviewers` — `graph` 유형일 때 사용하는 검토자 목록. 선택.
- `fan_out` — `swarm` 유형일 때 사용하는 병렬 분기 수. 선택.
- `peer_registration` — `agents-as-tools` 유형일 때 사용하는 피어 등록 설정. 선택.

`parent_pattern_id` 가 지정된 경우 데몬은 상위 패턴부터 루트까지의 깊이를 계산하고, `AutonomyPolicy` 의 `pattern_depth_cap`(기본값 3) 을 초과하면 요청을 거부한다(D24). 순환 참조(cycle)도 탐지해 차단한다. 새로 생성된 패턴의 라이프사이클은 항상 `pending` 으로 시작한다.

성공 응답: 생성된 패턴 객체. `pattern_created` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

**GET /patterns — 패턴 목록 조회**

쿼리 파라미터:

- `plan_id` — 조회할 플랜 식별자. 필수.

해당 플랜에 속한 패턴 배열을 반환한다.

## 권한 / 제약

- `GET` 은 `plan_id` 없이 호출하면 400 오류를 반환한다.
- `POST` 에서 `parent_pattern_id` 를 지정한 경우, 참조하는 상위 패턴이 존재해야 한다. 존재하지 않으면 404.
- 중첩 깊이 초과 또는 순환 참조 감지 시 400 오류를 반환한다.
- 패턴 생성 시점에는 `kind` 에 따른 형상 유효성 검사(shape validation)가 실행되지 않는다. 형상 검사는 `pending → active` 라이프사이클 전환 시점에 수행된다.

## provenance

D22 에서 협업 패턴이 7번째 1등급 엔티티로 확정되었다. D24 에서 `pattern_depth_cap` 을 AutonomyPolicy 에서 읽어 중첩 깊이를 제어하는 규칙이 결정되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/pattern.rs` 에 위치한다.

## 미확정 (OPEN)

- `short_code` 의 유일성 범위가 플랜 단위인지 프로젝트 단위인지 확인되지 않았다.
- `steps`, `reviewers`, `fan_out`, `peer_registration` 각 필드의 직렬화 형식(JSON 문자열 vs 구조화 객체)이 실제 구현에서 어떻게 저장되는지 확인이 필요하다.
- `pattern_created` SSE 이벤트의 채널 구독 방식(plan 단위 채널 vs 전역 채널)이 명시되지 않았다.
