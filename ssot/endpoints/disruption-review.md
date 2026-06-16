---
id: endpoint.disruption-review
kind: Endpoint
title: "GET /disruption-reviews/:id"
definition: >
  단일 혼란 검토(disruption review)를 조회하는 엔드포인트.
  상태, 해결 방식, 영향받는 시나리오 목록, 보충 메모를 포함한 전체 행을 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
relatesTo:
  - to: concept.disruption-review
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

특정 혼란 검토 한 건의 전체 내용을 조회하는 읽기 전용 엔드포인트다. 검토의 현재 상태(pending 또는 resolved), 해결 방식, 영향받는 시나리오 식별자 목록, 작성된 메모를 포함해 반환한다. 사람이 검토 내용을 확인하거나, 오케스트레이터가 게이트 해제 여부를 판단할 때 사용한다.

## 요청 / 응답

**GET /disruption-reviews/:id**

경로 파라미터:

- `id`: 조회할 혼란 검토의 식별자.

응답: 혼란 검토 행 전체. 다음 필드가 포함된다:

- `id`: 검토 식별자.
- `plan_id`: 이 검토가 속한 플랜.
- `source_kind` / `source_id`: 변경을 일으킨 원천의 종류와 식별자.
- `impacted_scenario_ids`: 영향받는 시나리오 식별자 목록.
- `status`: 현재 상태. `pending` 또는 `resolved`.
- `resolution`: 해결 방식 열거값. resolved 상태에서만 채워진다.
- `note`: 보충 메모.
- 생성·해결 시각 등 타임스탬프.

## 권한 / 제약

- 읽기 전용. 어떤 상태도 변경하지 않으며 SSE 이벤트를 발행하지 않는다.
- 존재하지 않는 `id`를 요청하면 오류를 반환한다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/disruption.rs`에서 유추됨.

## 미확정 (OPEN)

- 응답에 포함되는 정확한 필드 목록과 타임스탬프 컬럼명 미확인.
- `resolution` 열거값 전체 목록 미확인(spec에 retired / edited / kept_impacted가 있으나 이 엔드포인트의 응답 기술에서는 별도 확인 필요).
- 존재하지 않는 id 요청 시 반환 HTTP 상태 코드 미확인.
