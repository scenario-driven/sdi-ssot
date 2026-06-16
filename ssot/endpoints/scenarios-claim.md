---
id: endpoint.scenarios-claim
kind: Endpoint
title: "POST /scenarios/:id/claim"
definition: >
  D29: 시나리오의 claim_status를 active로 전환하는 엔드포인트. 시나리오에 선언된
  파일 경로 글로브 패턴이 현재 실행 세션에 의해 독점 클레임됨을 데몬에 등록한다.
  완료 시 SSE 이벤트를 발행한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario-claim
    type: mutates
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

SDI D29 파일 클레임 메커니즘의 일부로, 특정 시나리오가 선언하는 파일 경로 글로브 패턴을 현재 세션이 독점적으로 작업 중임을 데몬에 등록한다. `claim_status` 필드가 `active`로 전환된다.

호출자(클라이언트 또는 훅)는 이 엔드포인트를 호출하기 전에 `GET /scenarios/active-claims`를 먼저 조회하여 해당 파일 경로 글로브가 다른 세션과 겹치지 않는지 확인할 책임이 있다. 이 엔드포인트 자체는 중복 클레임을 자동으로 거부하지 않을 수 있다.

클레임이 완료되면 `claim_status_changed` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

- 메서드: POST
- 경로 파라미터: `:id` (시나리오 고유 식별자)
- 요청 본문: 없음 또는 세션 식별자 포함 가능(미확인)
- 응답: `claim_status`가 `active`로 갱신된 시나리오 전체 필드(또는 클레임 객체)
- 이미 claim_status가 active인 시나리오에 다시 호출 시의 동작 미확인
- 전환 완료 즉시 `claim_status_changed` SSE 이벤트 발행

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 중복 클레임 방지는 이 엔드포인트의 책임이 아니다. 호출 전 `GET /scenarios/active-claims`로 겹치는 경로가 없는지 확인하는 것은 호출자의 책임이다.
- retired 시나리오에 claim을 허용하는지는 미확인이다.
- 시나리오에 파일 경로 글로브 패턴이 선언되어 있지 않을 경우 claim의 의미가 불분명하므로, 해당 경우 거부 여부를 확인해야 한다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `POST /scenarios/:id/claim` 핸들러를 추론하여 작성하였다. D29 파일 클레임 설계와 `concept.scenario-claim` 노드, 그리고 `GET /scenarios/active-claims` 엔드포인트와의 협력 구조를 기반으로 작성하였다. SSE 이벤트 이름 `claim_status_changed` 는 release 엔드포인트와 공유되는 이벤트 이름으로 추론하였다.

## 미확정 (OPEN)

- 요청 본문에 세션 식별자(caller 정보)를 포함해야 하는지 미확인. 세션 식별자 없이 claim을 등록하면 누가 클레임했는지 추적하기 어렵다.
- 동일한 파일 경로 글로브를 가진 다른 시나리오가 이미 active 클레임 중일 때 이 엔드포인트가 충돌을 탐지하고 거부하는지, 아니면 호출자가 전적으로 책임지는지 미확인.
- 클레임 만료 시간(TTL)이 존재하는지, 세션이 비정상 종료될 경우 고아 클레임 정리 방법 미확인.
- `claim_status_changed` SSE 페이로드에 이전 상태와 새 상태 모두 포함되는지 미확인.
