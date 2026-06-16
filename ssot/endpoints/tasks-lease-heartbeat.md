---
id: endpoint.tasks-lease-heartbeat
kind: Endpoint
title: "POST /tasks/:id/lease/heartbeat"
definition: 현재 임대 보유자가 임대 유효 시간을 연장한다. 호출자가 현재 임대 보유자가 아니면 거부된다. 장시간 실행되는 에이전트가 임대를 유지하기 위해 주기적으로 호출한다.
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

임대를 보유한 에이전트가 작업이 아직 진행 중임을 알리고, 임대의 만료 시간을 현재 시점에서 다시 연장한다. TTL 만료 전에 이 엔드포인트를 반복적으로 호출함으로써 에이전트는 임대를 잃지 않고 장시간 작업을 안전하게 진행할 수 있다.

호출자가 요청 본문에 명시한 `holder` 가 현재 임대 보유자와 일치하지 않으면 400 오류로 거부된다. 이를 통해 임대를 보유하지 않은 에이전트가 다른 에이전트의 임대를 임의로 연장하는 것을 막는다.

## 요청 / 응답

**요청**
- 메서드: POST
- 경로 파라미터: `id` — 임대를 갱신할 태스크의 ID
- 요청 본문(JSON): `holder` (문자열, 필수) — 임대를 갱신하려는 보유자 식별자. 현재 임대 보유자와 일치해야 한다.
- TTL 연장 시간: 최초 임대 요청 시 지정한 `ttl_seconds` 와 동일한 값으로 갱신되는지, 또는 별도로 지정 가능한지 미확인.

**응답**
- 성공(200): 갱신된 임대 상태(새 만료 시각 포함) 반환.
- 보유자 불일치(400): 요청 본문의 `holder` 가 현재 임대 보유자와 다를 때 반환된다.
- 활성 임대 없음: 임대가 이미 만료되어 없는 경우의 처리 방식 미확인(400 또는 404 가능성).
- 태스크 없음(404): 지정한 태스크가 존재하지 않을 때.

## 권한 / 제약

- 현재 임대 보유자만 heartbeat 를 보낼 수 있다. 다른 에이전트는 보유자를 빼앗을 수 없다.
- 임대가 이미 만료된 경우에는 heartbeat 가 실패한다 — 만료 후 재획득은 `/tasks/:id/lease` POST 를 다시 호출해야 한다.
- 에이전트는 TTL 만료보다 충분히 앞서 heartbeat 를 보내야 한다. 권장 주기는 TTL 의 절반 이하다(미확인).

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- lease/heartbeat/release 세 엔드포인트가 임대 수명주기를 구성한다. heartbeat 는 임대를 유지하는 단계이며, 작업 완료 후에는 release 로 명시적으로 반납한다.

## 미확정 (OPEN)

- heartbeat 시 TTL 연장 시간이 최초 임대 요청의 `ttl_seconds` 와 동일한지, 아니면 요청 본문에서 별도로 지정할 수 있는지 미확인.
- 만료된 임대에 대한 heartbeat 응답 코드(400 vs 404 vs 409) 미확인.
- heartbeat 성공 시 SSE 이벤트 발행 여부 미확인.
