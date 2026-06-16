---
id: endpoint.scenarios-retire
kind: Endpoint
title: "POST /scenarios/:id/retire"
definition: >
  시나리오를 소프트 삭제(retired) 처리하는 엔드포인트. retired_at 타임스탬프를 기록하여
  비활성화하며, 기존 draft/confirmed 상태는 보존된다. 완료 시 SSE 이벤트를 발행한다.
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

시나리오를 더 이상 사용하지 않는 상태로 표시한다. 하드 삭제가 아닌 소프트 삭제 방식으로, `retired_at` 타임스탬프를 현재 시각으로 기록함으로써 비활성화된다.

retire 처리 시 시나리오가 기존에 갖고 있던 draft 또는 confirmed 상태는 그대로 보존된다. 이는 나중에 unretire 했을 때 정확히 이전 상태로 복원할 수 있도록 하기 위함이다.

retired 시나리오는 라운드 검증 작업 대상에서 제외된다. 즉, 라운드가 진행 중일 때 retired 시나리오는 검증 대상으로 선정되지 않는다. 전환 완료 시 `scenario.retired` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

- 메서드: POST
- 경로 파라미터: `:id` (시나리오 고유 식별자)
- 요청 본문: 없음
- 응답: `retired_at`이 기록된 시나리오 전체 필드
- 이미 retired 상태인 시나리오에 다시 호출할 경우의 동작은 미확인(멱등 처리 또는 409)
- 전환 완료 즉시 `scenario.retired` SSE 이벤트 발행

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 소프트 삭제이므로 데이터베이스에서 행이 제거되지 않는다. 이력 조회 및 복원이 가능하다.
- claim_status가 active(활성 클레임)인 시나리오를 retire 하는 것이 허용되는지, 아니면 먼저 release 해야 하는지 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `POST /scenarios/:id/retire` 핸들러를 추론하여 작성하였다. 소프트 삭제 구조(retired_at 타임스탬프, 상태 보존)는 SDI 플러그인 컨텍스트 및 unretire 엔드포인트와의 대칭적 설계에서 추론하였다. SSE 이벤트 이름 `scenario.retired` 는 데몬 이벤트 버스 패턴에서 추론한 값이다.

## 미확정 (OPEN)

- 이미 retired 시나리오에 다시 retire를 호출했을 때 멱등 200 반환인지, 409 반환인지 확인 필요.
- claim_status가 active인 시나리오를 retire 할 때 선행 release 요구 여부 미확인.
- retired 시나리오 목록을 별도로 조회하는 API(예: `GET /scenarios?retired=true`)가 존재하는지 미확인.
- `scenario.retired` SSE 페이로드 구조(시나리오 전체 포함 여부) 미확인.
