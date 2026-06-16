---
id: endpoint.runs
kind: Endpoint
title: "POST /runs, GET /runs"
definition: LLM 에이전트의 태스크 실행 시도(run)를 새로 시작하거나, 태스크 또는 세션 단위로 실행 기록 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.run
    type: mutates
  - to: concept.task
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

두 가지 동작을 담당하는 엔드포인트다.

`POST /runs`는 LLM 에이전트가 특정 태스크에 대해 실행을 시작할 때 그 시도를 기록하는 run 행을 생성한다. 하나의 태스크에는 여러 번의 run이 존재할 수 있으며, 각 run은 독립적인 실행 시도를 나타낸다. run은 시작(`run.started`)과 종료(`POST /runs/:id/finish`)가 분리되어 있다.

`GET /runs`는 특정 태스크(`task_id`) 또는 특정 세션(`session_id`)에 연결된 run 목록을 반환한다.

## 요청 / 응답

**POST /runs 요청 본문 (JSON)**

- `task_id` (필수) — 이 run이 수행하는 태스크의 식별자.
- `actor` (선택, 기본값 `"agent"`) — 실행 주체. 사람이 직접 실행하거나 특정 에이전트를 구분할 때 사용한다.
- `session_id` (선택) — 이 run이 속하는 세션 식별자. 세션 단위로 여러 run을 묶을 때 사용한다.
- `payload` (선택) — run 시작 시 전달되는 추가 데이터. 자유 형식 JSON.

**POST 응답**

생성된 run 행의 전체 데이터를 반환한다. 생성된 `id`와 `started_at` 타임스탬프가 포함된다.

**SSE 이벤트**

- `run.started` — run 행이 생성된 직후 발행된다.

**GET /runs 쿼리 파라미터**

- `task_id` (조건부 필수) — 조회할 태스크 식별자.
- `session_id` (조건부 필수) — 조회할 세션 식별자.

`task_id` 또는 `session_id` 중 하나는 반드시 전달해야 한다. 둘 다 생략하면 요청이 거부된다.

**GET 응답**

조건에 맞는 run 배열을 반환한다. 각 항목에는 id, task_id, actor, session_id, started_at, finished_at, result 등이 포함된다.

## 권한 / 제약

- `GET /runs`에서 `task_id`와 `session_id`가 모두 없으면 400을 반환한다. 전체 run 목록 무제한 조회는 허용되지 않는다.
- `task_id`가 존재하지 않는 태스크를 가리킬 경우의 처리는 구현에 따른다.
- 이미 완료된 태스크에 대한 새 run 생성이 허용되는지는 미확인이다.

## provenance

태스크 실행 이력을 추적하기 위한 엔드포인트다. 하나의 태스크가 여러 차례 시도(run)될 수 있다는 설계를 반영한다. run은 시작 시 이 엔드포인트로 생성하고, 완료 시 `POST /runs/:id/finish`로 종료 기록을 남기는 두 단계로 구성된다. `sdi-plugin/crates/daemon/src/router/run.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- `task_id`와 `session_id`를 동시에 전달했을 때 AND 조건인지 OR 조건인지 명시되지 않았다.
- `actor`에 허용되는 값이 열거형으로 제한되는지, 자유 문자열인지 미확인이다.
- `payload`의 스키마가 별도로 정의되어 있는지 미확인이다.
- `run.started` SSE 이벤트의 페이로드 구조가 명시되지 않았다.
- GET 응답의 정렬 기준(`started_at` 내림차순 등)이 명시되지 않았다.
