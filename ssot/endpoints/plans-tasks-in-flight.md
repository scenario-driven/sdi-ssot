---
id: endpoint.plans-tasks-in-flight
kind: Endpoint
title: "GET /plans/:plan_id/tasks/in-flight"
definition: 특정 플랜의 모든 라운드에 걸쳐 현재 in_progress 상태인 태스크 목록을 반환한다. 라운드 활성화 시 일시 중단·중단 대상 태스크 파악과 대시보드의 진행 중 작업 표시에 사용된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
    type: reads
    note: "status가 in_progress인 태스크를 조회한다."
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

주어진 플랜 ID에 속한 **모든 라운드**를 통틀어 현재 `in_progress` 상태인 태스크만을 모아 반환한다. 라운드 경계를 넘어 진행 중인 작업을 한눈에 볼 수 있도록 설계된 조회 전용 엔드포인트다.

이 엔드포인트는 두 가지 핵심 소비 시나리오를 가진다. 첫째, 새로운 라운드를 활성화할 때 현재 진행 중인 태스크를 파악하여 일시 중단(pause) 또는 중단(abort) 처리 대상을 결정하는 데 쓰인다. 둘째, 대시보드가 "지금 어떤 작업이 진행 중인가"를 사용자에게 보여주는 데 활용된다.

## 요청 / 응답

**요청**

- 경로 파라미터: `:plan_id` — 조회할 플랜의 ULID

**응답**

- 해당 플랜의 모든 라운드에 걸쳐 `in_progress` 상태인 태스크 목록을 반환한다.
- 각 태스크 항목에는 소속 라운드 정보가 포함될 것으로 추정된다.
- 플랜에 in_progress 태스크가 없으면 빈 목록을 반환한다.
- 존재하지 않는 plan_id는 404로 응답한다.

## 권한 / 제약

- 쓰기 작업이 없는 순수 조회 엔드포인트다.
- 조회 범위는 플랜 전체다 — 특정 라운드로 범위를 좁히는 필터를 지원하는지 여부는 미확인.

## provenance

SDI 플러그인 데몬의 태스크 라우터(`sdi-plugin/crates/daemon/src/router/task.rs`)에 구현되어 있다. 라운드 활성화 흐름에서 내부적으로 호출될 수 있으며, 대시보드 SPA도 이 엔드포인트를 폴링하거나 SSE 이벤트 후 재조회하는 방식으로 소비할 것으로 추정된다.

## 미확정 (OPEN)

- 응답 페이로드에 라운드 ID / 라운드 번호가 태스크별로 포함되는지 미확인.
- 페이지네이션 또는 정렬 파라미터 지원 여부 미확인.
- 라운드 활성화 흐름이 이 엔드포인트를 HTTP 클라이언트로 호출하는지, 아니면 동일 프로세스 내 함수 호출로 처리하는지 미확인.
