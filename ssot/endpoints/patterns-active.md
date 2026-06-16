---
id: endpoint.patterns-active
kind: Endpoint
title: "GET /patterns/active"
definition: 라이프사이클이 active 상태인 협업 패턴 전체를 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
relatesTo:
  - to: concept.collaboration-pattern
    type: reads
    note: "lifecycle=active 인 패턴만 필터링해 반환한다"
  - to: concept.pattern-lifecycle
    type: reads
    note: "active 상태가 조회 필터의 기준이 된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

라이프사이클이 `active` 인 협업 패턴 전체를 조회한다. 다중 에이전트 디스패치가 발생하기 전에 유효한 통치 패턴이 존재하는지 확인하는 용도로 쓰이며, 대시보드에서 현재 활성 패턴 수를 표시할 때도 사용한다.

## 요청 / 응답

**GET /patterns/active**

쿼리 파라미터:

- `project_id` — 선택. 지정하면 해당 프로젝트에 속한 active 패턴만 반환한다. 생략하면 데몬이 알고 있는 모든 active 패턴을 반환한다.

응답: `lifecycle=active` 인 패턴 객체의 배열. 각 패턴에는 `id`, `plan_id`, `short_code`, `kind`, `applies_to`, `scope_id`, `parent_pattern_id`, `depth` 가 포함된다.

## 권한 / 제약

- 읽기 전용 엔드포인트다. 상태 변경이 없다.
- PreToolUse 훅(D26)이 이 엔드포인트를 호출해 활성 패턴 없이 다중 에이전트 디스패치가 시작되는 것을 차단한다. 응답이 빈 배열이면 훅은 해당 디스패치를 거부할 수 있다.
- `project_id` 없이 호출하면 전체 active 패턴이 반환되므로 규모가 큰 운영 환경에서는 응답 크기를 고려해야 한다.

## provenance

D26 에서 PreToolUse 훅이 다중 에이전트 디스패치 전 active 패턴 존재 여부를 검사한다는 규칙이 확정되었다. 이 엔드포인트는 그 검사의 조회 수단으로 설계되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/pattern.rs` 에 위치한다.

## 미확정 (OPEN)

- `project_id` 필터가 플랜 → 프로젝트 역방향 조인으로 구현되는지, 아니면 패턴 테이블에 `project_id` 컬럼이 직접 존재하는지 확인이 필요하다.
- 대시보드가 이 엔드포인트를 폴링하는지 아니면 SSE 구독으로 실시간 갱신하는지 명시되지 않았다.
- PreToolUse 훅이 이 엔드포인트를 직접 HTTP 호출하는지, 데몬 내부 함수로 처리하는지 확인되지 않았다.
