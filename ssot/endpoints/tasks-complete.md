---
id: endpoint.tasks-complete
kind: Endpoint
title: "POST /tasks/:id/complete"
definition: 태스크를 완료(done)로 표시하되, 반드시 시나리오별 검증 결과(evidence)를 첨부해야 한다. 증거 없이는 완료 처리가 불가능하며, 태스크 상태 변경·증거 기록·라운드 결과 반영이 원자적으로 커밋된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
    type: mutates
    note: "태스크 상태를 in_progress에서 done으로 전환한다."
  - to: concept.task-evidence
    type: mutates
    note: "시나리오별 검증 결과를 evidence로 기록한다."
  - to: concept.scenario
    type: reads
    note: "각 evidence 항목이 참조하는 시나리오가 해당 태스크의 상위 시나리오로 존재하는지 검증한다."
  - to: concept.regression
    type: mutates
    note: "라운드 결과(round results)가 태스크 완료와 함께 원자적으로 갱신된다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

태스크를 완료 상태로 전환하는 전용 엔드포인트다. `/tasks/:id/status`와 달리 이 엔드포인트는 증거(evidence) 없이는 완료 처리를 허용하지 않는다. 각 증거 항목은 시나리오 ID(ULID 또는 short_code), 결과(passing / failing / blocked), 그리고 증거 참조(파일:라인, 테스트명, URL 중 하나)로 구성된다.

**핵심 게이트**: 증거 항목에 기재된 모든 시나리오는 해당 플랜에 실제로 존재해야 하며, 이 태스크의 상위(parent) 시나리오여야 한다. 존재하지 않거나 관계가 잘못된 시나리오 ID는 즉시 거부된다. 이는 유령(ghost) ID나 잘못 참조된 ID로 인해 검증 결과가 조용히 누락되는 상황을 방지하는 설계다.

모든 처리는 원자적이다 — 태스크 상태 변경, 증거 기록, 라운드 결과 반영이 하나의 트랜잭션으로 묶이며 전부 성공하거나 전부 실패한다.

## 요청 / 응답

**요청**

- 경로 파라미터: `:id` — 완료 처리할 태스크의 ULID
- 바디:
  - `evidence`: 증거 목록. 각 항목은 다음을 포함한다.
    - `scenario_id`: 시나리오의 ULID 또는 short_code
    - `result`: `passing`, `failing`, `blocked` 중 하나
    - `evidence_ref`: 파일 경로+라인, 테스트 이름, 또는 URL

**응답**

- 성공 시 완료 처리된 태스크와 기록된 evidence 목록을 반환한다.
- 태스크 상태 변경 완료 후 태스크별로 `task.completed` SSE, 시나리오별로 `round.result.updated` SSE가 발행된다.

**거부 조건**

- evidence 목록이 비어 있는 경우
- 참조된 시나리오 ID가 플랜에 존재하지 않는 경우
- 참조된 시나리오가 해당 태스크의 상위 시나리오가 아닌 경우
- `result` 값이 허용된 열거형 밖인 경우

## 권한 / 제약

- evidence 없는 done 전환은 이 엔드포인트에서도 허용되지 않는다.
- 시나리오 ID 검증은 플랜 범위 안에서 수행된다 — 다른 플랜의 시나리오를 참조하면 거부된다.
- 원자성 보장: 일부 evidence만 기록되고 태스크 상태가 변경되거나 그 반대가 되는 부분 커밋은 발생하지 않는다.

## provenance

SDI 플러그인 데몬의 태스크 라우터(`sdi-plugin/crates/daemon/src/router/task.rs`)에 구현되어 있다. 이 엔드포인트의 존재는 `/tasks/:id/status`에서 done 전환을 금지하는 결정과 쌍을 이룬다 — 두 엔드포인트가 함께 태스크 완료의 증거 필수 원칙을 강제한다.

## 미확정 (OPEN)

- `result: failing`인 evidence가 포함된 태스크를 완료 처리할 수 있는지, 아니면 모든 항목이 passing이어야 하는지 미확인.
- 부분 증거(일부 상위 시나리오만 포함)가 허용되는지, 아니면 모든 상위 시나리오에 대한 verdict가 필수인지 미확인.
- SSE 이벤트가 데몬 내부 버스를 통해 발행되는지 아니면 별도 채널을 사용하는지 미확인.
