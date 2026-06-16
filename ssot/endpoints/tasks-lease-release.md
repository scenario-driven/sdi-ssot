---
id: endpoint.tasks-lease-release
kind: Endpoint
title: "POST /tasks/:id/lease/release"
definition: 임대 보유자가 임대를 명시적으로 반납한다. 호출자가 현재 임대 보유자인 경우에만 해제되며, 아닌 경우 released:false 를 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.task
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

태스크에 대한 임대를 보유한 에이전트가 작업을 마치거나 중단할 때 임대를 명시적으로 반납한다. 요청 본문의 `holder` 가 현재 임대 보유자와 일치하면 임대를 즉시 해제하고 `released: true` 를 반환한다. 일치하지 않으면 임대를 해제하지 않고 `released: false` 를 반환한다.

이 엔드포인트는 임대가 TTL 만료로 자동 해제되길 기다리지 않고, 작업 완료 즉시 다른 에이전트가 임대를 획득할 수 있도록 빠르게 반납하기 위해 사용된다.

## 요청 / 응답

**요청**
- 메서드: POST
- 경로 파라미터: `id` — 임대를 반납할 태스크의 ID
- 요청 본문(JSON): `holder` (문자열, 필수) — 임대를 반납하려는 보유자 식별자

**응답**
- 성공(200): `released` (boolean) 반환.
  - `released: true` — 호출자가 보유자와 일치하여 임대가 해제됨.
  - `released: false` — 호출자가 현재 임대 보유자가 아니어서 해제하지 않음.
- 태스크 없음(404): 지정한 태스크가 존재하지 않을 때.

## 권한 / 제약

- 임대 해제 권한은 현재 보유자에게만 있다. 다른 에이전트는 임의로 다른 에이전트의 임대를 해제할 수 없다.
- 임대가 이미 만료되었거나 존재하지 않는 상태에서 release 를 호출했을 때의 응답(released:false 반환 vs 다른 오류) 미확인.
- 에이전트는 작업 완료 후 반드시 release 를 호출해 다음 에이전트가 지체 없이 임대를 획득할 수 있도록 해야 한다. TTL 만료에만 의존하면 다음 작업이 최대 TTL 시간만큼 지연될 수 있다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- lease/heartbeat/release 세 엔드포인트가 임대 수명주기를 구성한다. release 는 임대를 정상적으로 종료하는 마지막 단계다. 비정상 종료(에이전트 크래시 등)에는 TTL 만료가 안전망 역할을 한다.

## 미확정 (OPEN)

- 임대가 없는 상태에서 release 를 호출했을 때 `released: false` 를 반환하는지, 404 를 반환하는지 미확인.
- release 성공 시 SSE 이벤트 발행 여부 미확인.
- `released: false` 반환 시 HTTP 상태 코드가 200인지 400인지 미확인.
