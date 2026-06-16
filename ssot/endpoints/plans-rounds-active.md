---
id: endpoint.plans-rounds-active
kind: Endpoint
title: "GET /plans/:plan_id/rounds/active"
definition: 특정 플랜에서 현재 활성 상태인 라운드를 조회하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
    type: reads
  - to: concept.plan
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

주어진 플랜에서 현재 `active` 상태인 라운드를 단 하나 반환한다. 한 플랜에 동시에 활성화된 라운드는 최대 하나이므로, 이 엔드포인트는 항상 단일 객체 또는 null을 반환한다.

CLI와 대시보드가 현재 작업 중인 라운드 컨텍스트를 결정할 때 이 엔드포인트를 사용한다.

## 요청 / 응답

**GET /plans/:plan_id/rounds/active**

경로 파라미터:

- `plan_id` (필수): 조회할 플랜의 식별자.

응답(JSON):

- 활성 라운드가 존재하면: 라운드 객체 전체(id, short_code, round_number, mode, in_flight_policy, disruption_policy, status, created_at, updated_at 포함).
- 활성 라운드가 없으면: `null`.

## 권한 / 제약

- 존재하지 않는 `plan_id`를 지정하면 404를 반환한다.
- 읽기 전용 엔드포인트이며 상태를 변경하지 않는다.
- 응답이 `null`인 것은 오류가 아니다. 플랜에 아직 라운드가 없거나 모든 라운드가 `completed`인 정상 상태다.

## provenance

이 엔드포인트는 CLI와 대시보드가 "지금 어떤 라운드에서 작업 중인가"를 빠르게 파악하기 위한 조회 단축 경로다. `GET /rounds?plan_id=...`로 목록을 가져와 `active` 항목을 찾는 것과 동등하지만, 단일 객체로 바로 반환하므로 클라이언트 측 필터링이 불필요하다.

## 미확정 (OPEN)

- 응답이 `null`일 때 HTTP 상태 코드를 200(null 본문)으로 내려주는지, 아니면 404로 내려주는지 미확인.
- 응답 래핑 구조(`{ "round": null }` vs 직접 `null`) 미확인.
