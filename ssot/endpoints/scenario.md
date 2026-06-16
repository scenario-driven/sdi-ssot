---
id: endpoint.scenario
kind: Endpoint
title: "GET /scenarios/:id, PUT /scenarios/:id"
definition: >
  단일 시나리오를 조회하거나, 그 시나리오의 Given / When / Then 본문을 갱신하는 엔드포인트.
  조회는 읽기 전용이며, 갱신 시에는 세 필드 모두 함께 전송해야 한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario
    type: mutates
  - to: concept.given-when-then
    type: mutates
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

시나리오 하나를 식별자(`:id`)로 지정하여 조회하거나 갱신하는 엔드포인트 쌍이다.

**GET** 는 해당 시나리오의 현재 상태 전체(식별자, 제목, Given / When / Then, 상태, 작성 시각 등)를 반환한다. 변경 없이 내용을 확인할 때 사용한다.

**PUT** 는 시나리오의 행동 명세인 Given(사전 조건) / When(행동 트리거) / Then(기대 결과)을 덮어쓴다. 세 필드 모두 요청 본문에 포함되어야 하며, 하나라도 빠지면 요청이 거부된다. 갱신이 완료되면 `scenario.updated` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

**GET /scenarios/:id**

- 요청 파라미터: 경로 내 `:id` (시나리오 고유 식별자)
- 요청 본문: 없음
- 응답: 시나리오 전체 필드(식별자, 제목, Given / When / Then 텍스트, 상태, 확정 여부, retired_at, claim_status, 생성 시각, 수정 시각 등)
- 해당 id가 존재하지 않으면 404 반환

**PUT /scenarios/:id**

- 요청 파라미터: 경로 내 `:id`
- 요청 본문(JSON): `given`, `when`, `then` 세 필드 필수
- 응답: 갱신된 시나리오 전체 필드
- 갱신 직후 `scenario.updated` SSE 이벤트 발행
- 세 필드 중 하나라도 누락되면 400 반환

## 권한 / 제약

- 인증 토큰 없이 로컬 데몬에 직접 접근하는 구조이므로 별도 인증 계층은 없다.
- retired 상태인 시나리오도 조회와 갱신 모두 허용된다. 단, retired 시나리오는 라운드 검증 작업 대상에서 제외된다.
- PUT는 Given / When / Then 세 필드만 변경한다. 시나리오의 상태(draft / confirmed), claim_status, retired_at 등은 이 엔드포인트로 변경할 수 없다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `GET /scenarios/:id` 와 `PUT /scenarios/:id` 핸들러를 추론하여 작성하였다. SSE 이벤트 이름 `scenario.updated` 는 데몬 이벤트 버스 패턴에서 추론한 값이다.

## 미확정 (OPEN)

- PUT 요청 시 `title`(제목) 필드도 함께 갱신할 수 있는지, 아니면 제목은 별도 엔드포인트에서만 변경 가능한지 확인 필요.
- retired 시나리오에 PUT를 시도했을 때 경고 없이 허용하는지, 아니면 409 등의 응답을 반환하는지 미확인.
- SSE 이벤트 페이로드에 변경 전 / 후 필드 차이가 포함되는지, 아니면 갱신된 전체 시나리오만 포함되는지 미확인.
