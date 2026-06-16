---
id: endpoint.rounds
kind: Endpoint
title: "POST /rounds, GET /rounds"
definition: 라운드를 생성하거나 특정 플랜에 속한 라운드 목록을 조회하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
    type: mutates
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

라운드는 플랜 안에서 시나리오 검증 주기를 하나 단위로 묶는 개념이다. 이 엔드포인트는 두 가지 역할을 한다. POST는 새 라운드를 만들고, GET은 특정 플랜에 속한 라운드 전체를 나열한다.

라운드 번호는 플랜 안에서 자동으로 순서에 따라 부여된다. 사람이 직접 번호를 지정하지 않는다.

POST 요청에 패턴이 지정되어 있지 않으면 D23 결정에 따라 직접 센티넬(direct sentinel)이 적용된다.

라운드가 생성되면 `round.created` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /rounds**

요청 본문(JSON):

- `plan_id` (필수): 이 라운드가 속할 플랜의 식별자.
- `short_code` (필수): 라운드를 식별하는 짧은 코드.
- `mode` (선택, 기본값 `strict-regression`): 회귀 검증 모드. 현재 지원 값은 `strict-regression`.
- `in_flight_policy` (선택, 기본값 `pause`): 라운드 활성화 시 진행 중인 태스크를 어떻게 처리할지 결정하는 정책. `pause`, `abort`, `continue` 중 하나.
- `disruption_policy` (선택, 기본값 `needs-review`): 시나리오가 영향을 받았을 때의 처리 정책. `needs-review` 등.

응답: 생성된 라운드 객체. 자동 부여된 라운드 번호, 상태(`planning`), 생성 시각 포함.

**GET /rounds**

쿼리 파라미터:

- `plan_id` (필수): 조회할 플랜의 식별자.

응답: 해당 플랜에 속한 라운드 배열. 각 항목에 라운드 번호, 모드, 상태, 타임스탬프 포함.

## 권한 / 제약

- GET 시 `plan_id` 쿼리 파라미터가 없으면 요청이 거부된다.
- 라운드 번호는 자동 부여이며 외부에서 덮어쓸 수 없다.
- 패턴이 지정되지 않으면 직접 센티넬로 대체된다(D23).

## provenance

라운드 생성 및 목록 조회 동작은 D6(strict-regression 기본값), D23(직접 센티넬 폴백) 결정에 근거한다. `in_flight_policy` 기본값 `pause` 및 `disruption_policy` 기본값 `needs-review`는 안전 우선 원칙에서 비롯된다.

## 미확정 (OPEN)

- `mode` 값으로 `strict-regression` 외에 추가 모드가 존재하는지 미확인.
- SSE 이벤트 `round.created` 의 페이로드 스키마 세부 사항 미확인.
- `disruption_policy` 지원 값의 전체 목록 미확인.
