---
id: endpoint.plans-disruption-reviews
kind: Endpoint
title: "GET /plans/:plan_id/disruption-reviews"
definition: 특정 플랜에 속한 중단 영향 검토(disruption review) 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
relatesTo:
  - to: concept.disruption-review
    type: reads
    note: "플랜 단위로 등록된 중단 영향 검토 레코드를 반환한다"
  - to: concept.plan
    type: reads
    note: "plan_id 로 조회 범위를 한정한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

플랜에 연결된 중단 영향 검토 항목들을 나열한다. 중단 영향 검토는 진행 중인 라운드나 작업이 예상치 못한 방해 요인에 의해 영향을 받을 때 생성되는 검토 레코드다. 라운드 활성화 로직이 미해결 검토가 남아있는지 확인하는 데 사용하며, 대시보드는 이를 통해 미결 영향 검토 건수를 표시한다.

## 요청 / 응답

**GET /plans/:plan_id/disruption-reviews**

경로 파라미터:

- `plan_id` — 조회할 플랜 식별자. 필수.

쿼리 파라미터:

- `status` — 선택. `pending` 또는 `resolved` 를 지정해 상태별로 필터링한다. 생략하면 모든 상태의 검토 항목을 반환한다.

응답: 해당 플랜에 속한 중단 영향 검토 객체의 배열. 각 항목에는 검토 식별자, 연결된 엔티티 정보, 현재 상태, 생성 시각이 포함된다.

## 권한 / 제약

- 읽기 전용 엔드포인트다. 상태 변경이 없다.
- 지정한 `plan_id` 에 해당하는 플랜이 존재하지 않으면 404 를 반환한다.
- 라운드 활성화 로직은 `status=pending` 으로 필터링한 결과가 비어있지 않으면 라운드 시작을 보류할 수 있다.

## provenance

라운드 활성화 전 미해결 중단 영향 검토 확인 요건에서 이 엔드포인트의 필요성이 도출되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/disruption.rs` 에 위치한다.

## 미확정 (OPEN)

- 중단 영향 검토 레코드가 어떤 엔티티(라운드, 시나리오, 태스크 등)에 연결될 수 있는지 전체 앵커 목록이 명시되지 않았다.
- 라운드 활성화 로직이 이 엔드포인트를 직접 HTTP 호출하는지, 데몬 내부에서 동일한 쿼리를 수행하는지 확인이 필요하다.
- `pending` 검토가 있을 때 라운드 활성화를 완전히 차단하는지, 경고만 발생시키는지 정책이 확인되지 않았다.
- 검토를 `resolved` 로 전환하는 별도 엔드포인트의 존재 여부 및 위치가 명시되지 않았다.
