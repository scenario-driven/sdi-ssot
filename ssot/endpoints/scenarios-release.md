---
id: endpoint.scenarios-release
kind: Endpoint
title: "POST /scenarios/:id/release"
definition: >
  D29: 시나리오의 claim_status를 released로 전환하는 엔드포인트. active 클레임을
  해제하여 해당 파일 경로 글로브 패턴을 다른 시나리오나 세션이 사용할 수 있도록 해방한다.
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

SDI D29 파일 클레임 메커니즘의 일부로, `POST /scenarios/:id/claim`으로 등록한 독점 파일 경로 클레임을 해제한다. `claim_status` 필드가 `released`로 전환된다.

클레임이 해제되면 해당 시나리오가 선언하고 있던 파일 경로 글로브 패턴이 다른 시나리오나 다른 세션에서 자유롭게 클레임할 수 있는 상태가 된다. `GET /scenarios/active-claims`의 결과에서도 이 시나리오의 항목이 사라진다.

작업이 정상 완료된 후 또는 세션이 중단될 때 호출하여 고아 클레임이 남지 않도록 한다. 전환 완료 시 `claim_status_changed` SSE 이벤트가 구독 중인 클라이언트에게 전송된다.

## 요청 / 응답

- 메서드: POST
- 경로 파라미터: `:id` (시나리오 고유 식별자)
- 요청 본문: 없음
- 응답: `claim_status`가 `released`로 갱신된 시나리오 전체 필드(또는 클레임 객체)
- claim_status가 이미 released 이거나 처음부터 active가 아닌 경우의 동작 미확인
- 전환 완료 즉시 `claim_status_changed` SSE 이벤트 발행

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 이 엔드포인트는 claim을 등록한 세션 외의 다른 세션이 호출하는 것이 허용되는지 미확인이다. 세션 식별자 검증이 없으면 타 세션이 강제로 클레임을 해제할 수 있다.
- 클레임이 active가 아닌 상태에서 release를 호출했을 때 오류를 반환하는지, 멱등 처리하는지 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `POST /scenarios/:id/release` 핸들러를 추론하여 작성하였다. claim 엔드포인트와 대칭적 설계로 동작하며, D29 파일 클레임 메커니즘 전반의 설계에서 도출하였다. SSE 이벤트 이름 `claim_status_changed` 는 claim 엔드포인트와 공유되는 이름으로 추론하였다.

## 미확정 (OPEN)

- claim을 등록한 세션과 다른 세션이 release를 호출할 수 있는지, 세션 소유권 검증이 있는지 미확인.
- claim_status가 active가 아닌 상태에서 호출 시 멱등 200 반환인지, 409/400 반환인지 확인 필요.
- 세션 비정상 종료 시 active 클레임의 자동 해제 방법(TTL, 감시 메커니즘 등) 미확인.
- `claim_status_changed` SSE 페이로드에 이전 상태(`active`)와 새 상태(`released`) 모두 포함되는지 미확인.
