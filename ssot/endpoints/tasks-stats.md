---
id: endpoint.tasks-stats
kind: Endpoint
title: "GET /tasks/stats"
definition: 태스크를 상태별로 집계한 카운트를 반환한다. 선택적으로 특정 프로젝트만 대상으로 범위를 좁힐 수 있으며, 대시보드와 Prometheus 메트릭 수집에 사용된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.task
    type: reads
  - to: concept.usage-budget
    type: relates-to
    note: "태스크 상태별 카운트는 작업 진행률·사용량 메트릭 산출의 기반 데이터로 활용된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

데몬이 관리하는 태스크 전체(또는 특정 프로젝트)를 상태별로 집계해 카운트를 반환한다. 집계 대상 상태는 `todo`, `in_progress`, `blocked`, `done`, `cancelled` 다섯 가지다. `project_id` 쿼리 파라미터를 제공하면 해당 프로젝트 소속 태스크만 집계하고, 생략하면 데몬 전체 범위의 태스크를 집계한다.

대시보드에서 현재 작업 현황을 한눈에 파악하거나, Prometheus 등 모니터링 도구가 주기적으로 메트릭을 수집할 때 이 엔드포인트를 사용한다.

## 요청 / 응답

**요청**
- 메서드: GET
- 경로 파라미터: 없음
- 쿼리 파라미터: `project_id` (문자열, 선택) — 특정 프로젝트로 집계 범위를 좁힌다. 생략하면 데몬 전체 범위.
- 요청 본문: 없음

**응답**
- 성공(200): 상태별 카운트 객체. 예시 구조:
  - `todo`: 시작 전 태스크 수
  - `in_progress`: 진행 중 태스크 수
  - `blocked`: 차단된 태스크 수
  - `done`: 완료된 태스크 수
  - `cancelled`: 취소된 태스크 수
- 잘못된 project_id(404 또는 빈 카운트 반환): `project_id` 가 존재하지 않는 프로젝트를 가리킬 때의 처리 방식 미확인.

## 권한 / 제약

- 읽기 전용 엔드포인트. 태스크 상태를 변경하지 않는다.
- `project_id` 를 생략하면 데몬이 알고 있는 모든 프로젝트의 태스크를 합산한 전체 카운트를 반환한다.
- Prometheus 스크래핑 등 자동화된 메트릭 수집 클라이언트가 주기적으로 폴링하도록 설계되었다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- 작업 진행 상황을 집계 수준에서 파악할 수 있는 요약 엔드포인트로, 개별 태스크 목록 조회 없이 전체 진행률을 빠르게 확인하기 위한 용도다. 사용량 예산(`usage-budget`) 추적과도 연결된다 — 태스크 카운트는 진행률·완료율 메트릭의 기반 데이터가 된다.

## 미확정 (OPEN)

- 응답 구조가 플랫 객체(상태명: 카운트)인지, 배열 형태인지 구현 확인 전까지 미검증 상태다.
- 존재하지 않는 `project_id` 를 지정했을 때 404 오류를 반환하는지, 아니면 모든 카운트가 0인 정상 응답을 반환하는지 미확인.
- 상태 이름의 정확한 열거 목록 및 철자(예: `in_progress` vs `inProgress`) 구현 확인 필요.
- Prometheus exposition 형식(`text/plain; version=0.0.4`)을 직접 지원하는지, 아니면 JSON 만 반환하는지 미확인.
