---
id: endpoint.tasks-lease
kind: Endpoint
title: "POST /tasks/:id/lease, GET /tasks/:id/lease"
definition: 태스크에 대한 시한부 독점 임대(lease)를 획득(POST)하거나 현재 임대 상태를 조회(GET)한다. 이미 다른 보유자가 임대 중이면 새 획득 요청은 거부된다.
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

하나의 경로(`:id`)에 두 가지 메서드를 제공하는 복합 엔드포인트다.

**POST**: 특정 태스크에 대한 시한부 독점 임대를 요청한다. 요청 본문에 임대를 보유할 주체 이름(holder)과 유효 시간(ttl_seconds, 기본값 120초)을 지정한다. 해당 태스크에 현재 유효한 임대가 없으면 임대를 부여하고 `acquired: true` 를 반환한다. 이미 다른 보유자가 임대 중이면 덮어쓰지 않고 `acquired: false` 와 현재 임대 상태를 반환한다. 이 메커니즘은 여러 에이전트가 동시에 같은 태스크를 작업하지 않도록 상호 배제를 제공한다.

**GET**: 현재 임대 상태를 조회한다. 임대를 획득하지 않고 누가 임대 중인지, 언제 만료되는지만 확인하고 싶을 때 사용한다.

## 요청 / 응답

**POST 요청**
- 경로 파라미터: `id` — 임대를 요청할 태스크의 ID
- 요청 본문(JSON): `holder` (문자열, 필수) — 임대 보유자 식별자; `ttl_seconds` (정수, 선택, 기본값 120)

**POST 응답**
- 성공(200): `acquired` (boolean) — 임대 획득 여부. 현재 임대 상태(holder, 만료 시각) 포함. `acquired: false` 일 때도 200으로 응답하며 현재 임대 상태가 함께 반환된다.
- 태스크 없음(404): 지정한 태스크가 존재하지 않을 때.

**GET 요청**
- 경로 파라미터: `id` — 임대 상태를 조회할 태스크의 ID
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**GET 응답**
- 성공(200): 현재 임대 상태. 임대가 없으면 빈 상태(holder: null 또는 임대 없음 표시)를 반환한다.
- 태스크 없음(404): 지정한 태스크가 존재하지 않을 때.

## 권한 / 제약

- 임대는 선착순(first-writer-wins) 방식이다. 먼저 임대를 획득한 보유자의 임대가 유효한 동안 다른 보유자의 획득 요청은 거부된다(`acquired: false`).
- TTL 이 만료된 임대는 자동으로 해제된 것으로 간주되어 다음 획득 요청이 성공한다.
- 임대 갱신은 `/tasks/:id/lease/heartbeat`, 명시적 해제는 `/tasks/:id/lease/release` 를 사용한다.
- `ttl_seconds` 기본값은 120초이며, 장시간 작업 시 heartbeat 로 주기적으로 갱신해야 한다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- 여러 LLM 에이전트가 동시에 동일 태스크를 수정하는 경쟁 상태를 방지하기 위한 낙관적 잠금 메커니즘이다. lease/heartbeat/release 세 엔드포인트가 함께 임대 수명주기를 구성한다.

## 미확정 (OPEN)

- `holder` 필드의 형식(에이전트 이름 자유 문자열인지 구조화된 ID인지) 미확인.
- TTL 최대값 또는 최솟값 제한 존재 여부 미확인.
- 만료된 임대가 데이터베이스에서 즉시 제거되는지 아니면 만료 표시만 되는지 미확인.
- POST 성공 시 SSE 이벤트 발행 여부 미확인.
