---
id: endpoint.round
kind: Endpoint
title: "GET /rounds/:id"
definition: 특정 라운드 하나의 상세 정보를 조회하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
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

라운드 ID를 경로 파라미터로 받아 해당 라운드의 전체 정보를 돌려준다. 라운드의 현재 상태, 검증 모드, 각종 정책 설정, 그리고 생성·수정 시각을 포함한다.

## 요청 / 응답

**GET /rounds/:id**

경로 파라미터:

- `id` (필수): 조회할 라운드의 식별자.

응답(JSON):

- `id`: 라운드 식별자.
- `plan_id`: 이 라운드가 속한 플랜 식별자.
- `short_code`: 라운드의 짧은 코드.
- `round_number`: 플랜 내 자동 부여된 순번.
- `mode`: 검증 모드 (`strict-regression` 등).
- `in_flight_policy`: 진행 중 태스크 처리 정책 (`pause`, `abort`, `continue` 등).
- `disruption_policy`: 시나리오 영향 처리 정책 (`needs-review` 등).
- `status`: 라운드 현재 상태 (`planning`, `active`, `completed`).
- `created_at`: 생성 시각.
- `updated_at`: 마지막 수정 시각.

## 권한 / 제약

- 존재하지 않는 ID를 조회하면 404를 반환한다.
- 읽기 전용 엔드포인트이며 상태를 변경하지 않는다.

## provenance

라운드 상태 전이(`planning` → `active` → `completed`)는 별도의 활성화·완료 엔드포인트에서 다룬다. 이 엔드포인트는 순수 조회만 담당한다.

## 미확정 (OPEN)

- `status` 값의 전체 열거 목록 미확인 (이 외에 `cancelled` 같은 상태가 있는지).
- `round_number`가 응답 필드에 포함되는지, 아니면 `short_code`가 그 역할을 대신하는지 미확인.
