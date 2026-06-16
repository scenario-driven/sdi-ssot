---
id: endpoint.runs-finish
kind: Endpoint
title: "POST /runs/:id/finish"
definition: 실행 중인 run의 최종 상태를 기록하고 종료한다. 성공, 실패, 오류, 취소 중 하나의 결과를 남긴다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.run
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

진행 중인 run에 종료 상태를 기록하는 엔드포인트다. `POST /runs`로 시작된 run은 이 엔드포인트가 호출될 때까지 미완료 상태로 남아 있다. 이 엔드포인트를 호출하면 `finished_at` 타임스탬프가 기록되고, `result` 필드가 설정되며, `run.finished` SSE 이벤트가 발행된다.

run의 생애 주기는 `POST /runs`(시작) → `POST /runs/:id/finish`(종료)의 두 단계로 구성된다.

## 요청 / 응답

**경로 파라미터**

- `:id` — 종료할 run의 식별자.

**요청 본문 (JSON)**

- `result` (필수) — run의 최종 결과. 다음 네 가지 값 중 하나:
  - `passed` — 태스크가 성공적으로 완료되었다.
  - `failed` — 태스크가 실패했다(예상 가능한 실패, 검증 불통과 등).
  - `error` — 예상치 못한 오류로 run이 중단되었다.
  - `cancelled` — 외부 요인에 의해 run이 취소되었다.
- `notes` (선택) — 결과에 대한 부가 설명. 실패 원인, 에이전트 메모, 다음 시도를 위한 힌트 등을 자유 형식으로 기록한다.

**응답**

갱신된 run 행의 전체 데이터를 반환한다. `finished_at`과 `result`가 채워진 상태다.

**SSE 이벤트**

- `run.finished` — 종료 처리가 완료된 직후 발행된다.

## 권한 / 제약

- `result`는 `passed`, `failed`, `error`, `cancelled` 중 하나여야 한다. 그 외 값은 거부된다(400).
- 이미 종료된 run에 다시 finish를 호출할 경우의 동작(오류 반환 또는 덮어쓰기)은 미확인이다.
- 존재하지 않는 `:id`에 대한 요청은 404를 반환한다.

## provenance

`POST /runs`(endpoint.runs)와 쌍을 이루는 종료 엔드포인트다. run 기록이 종료 상태로 전환되어야 태스크 수준의 실행 이력이 완전한 형태가 된다. `run.finished` SSE를 통해 대시보드나 다른 구독자가 실시간으로 run 완료를 감지할 수 있다. `sdi-plugin/crates/daemon/src/router/run.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- 이미 `finished_at`이 설정된 run에 대해 재호출 시 오류를 반환하는지, 마지막 결과로 덮어쓰는지 명시되지 않았다.
- `run.finished` SSE 이벤트의 페이로드에 run 전체 정보가 포함되는지, id만 포함되는지 미확인이다.
- `result=failed`와 `result=error`의 의미론적 구분이 태스크 완료 상태에 어떤 영향을 미치는지(태스크 자동 실패 전환 여부 등) 미확인이다.
