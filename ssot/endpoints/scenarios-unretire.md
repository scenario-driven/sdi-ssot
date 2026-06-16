---
id: endpoint.scenarios-unretire
kind: Endpoint
title: "POST /scenarios/:id/unretire"
definition: >
  retire 처리된 시나리오를 원래 상태로 되돌리는 엔드포인트. retired_at 타임스탬프를
  지워 비활성화를 해제하며, 시나리오는 retire 이전의 draft/confirmed 상태로 복귀한다.
  완료 시 SSE 이벤트를 발행한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario
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

`POST /scenarios/:id/retire` 동작을 역전시킨다. `retired_at` 타임스탬프를 지워 시나리오를 다시 활성 상태로 만든다.

retire 시 draft 또는 confirmed 상태가 그대로 보존되어 있었으므로, unretire 후에는 retire 직전의 상태로 정확히 복귀한다. 별도의 상태 복구 로직이 필요하지 않다.

unretire된 시나리오는 다시 라운드 검증 대상에 포함될 수 있으며, confirmed 상태라면 D8 플랜 승인 게이트 집계에도 재참여한다. 전환 완료 시 `scenario.unretired` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

- 메서드: POST
- 경로 파라미터: `:id` (시나리오 고유 식별자)
- 요청 본문: 없음
- 응답: `retired_at`이 null로 초기화된 시나리오 전체 필드
- retired 상태가 아닌 시나리오에 호출할 경우의 동작은 미확인(멱등 처리 또는 409)
- 전환 완료 즉시 `scenario.unretired` SSE 이벤트 발행

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 이 엔드포인트는 `retired_at` 필드만 초기화한다. draft/confirmed 상태나 Given / When / Then 본문은 건드리지 않는다.
- unretire 후 시나리오가 어떤 라운드에 즉시 반영될지는 라운드 진행 상황에 따라 달라진다. 진행 중인 라운드의 검증 범위에 즉시 포함되는지, 다음 라운드부터 포함되는지 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `POST /scenarios/:id/unretire` 핸들러를 추론하여 작성하였다. retire 엔드포인트와 대칭적 설계로 동작하며, 상태 보존 후 복원 패턴은 retire 엔드포인트 정의에서 도출하였다. SSE 이벤트 이름 `scenario.unretired` 는 데몬 이벤트 버스 패턴에서 추론한 값이다.

## 미확정 (OPEN)

- retire 상태가 아닌 시나리오에 unretire를 호출했을 때 멱등 200 반환인지, 409 반환인지 확인 필요.
- unretire 후 현재 진행 중인 라운드의 검증 범위에 즉시 포함되는지 미확인.
- `scenario.unretired` SSE 페이로드 구조(시나리오 전체 포함 여부) 미확인.
