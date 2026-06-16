---
id: endpoint.plans-tasks-backlog
kind: Endpoint
title: "GET /plans/:plan_id/tasks/backlog"
definition: 특정 플랜의 모든 라운드에 걸쳐 아직 시작되지 않은(todo) 태스크 목록을 반환한다. 대시보드가 아직 처리되지 않은 분해된 작업들을 표시하는 데 사용된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
    type: reads
    note: "status가 todo인 태스크를 조회한다."
  - to: concept.plan
    type: reads
    note: "조회 범위를 특정 플랜으로 한정한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

주어진 플랜 ID에 속한 **모든 라운드**를 통틀어 아직 착수되지 않은(`todo`) 태스크 목록을 반환한다. 백로그(backlog)는 시나리오 분해(decomposition)를 통해 생성된 태스크 중 아직 `in_progress`로 전환되지 않은 대기 작업들의 집합이다.

대시보드가 "앞으로 해야 할 분해된 작업이 무엇인가"를 사용자에게 보여주는 주요 소비 시나리오다. 플랜 전체 범위에서 백로그를 볼 수 있어, 라운드별로 흩어진 대기 태스크를 한 화면에서 파악할 수 있다.

## 요청 / 응답

**요청**

- 경로 파라미터: `:plan_id` — 조회할 플랜의 ULID

**응답**

- 해당 플랜의 모든 라운드에 걸쳐 `todo` 상태인 태스크 목록을 반환한다.
- 플랜에 백로그 태스크가 없으면 빈 목록을 반환한다.
- 존재하지 않는 plan_id는 404로 응답한다.

## 권한 / 제약

- 쓰기 작업이 없는 순수 조회 엔드포인트다.
- `cancelled` 상태의 태스크는 백로그에 포함되지 않는다.

## provenance

SDI 플러그인 데몬의 태스크 라우터(`sdi-plugin/crates/daemon/src/router/task.rs`)에 구현되어 있다. `endpoint.plans-tasks-in-flight`와 대칭적으로 설계되어 있으며, 두 엔드포인트를 합치면 플랜 내 모든 활성·대기 작업의 전체 현황을 파악할 수 있다.

## 미확정 (OPEN)

- 응답에 각 태스크가 속한 라운드 정보가 포함되는지 미확인.
- `blocked` 상태 태스크가 이 엔드포인트에 포함되는지 아니면 별도 엔드포인트로 분리되는지 미확인.
- 특정 라운드 ID로 범위를 좁히는 쿼리 파라미터 지원 여부 미확인.
- 태스크 정렬 기준(생성 순, 우선순위 등) 미확인.
