---
id: endpoint.tasks-usage-preflight
kind: Endpoint
title: "GET /tasks/:id/usage/preflight"
definition: 태스크 실행 전에 과거 사용 기록을 기반으로 예상 비용을 휴리스틱으로 추정하여 반환하는 엔드포인트. 에이전트가 태스크 시작 전 모델 티어를 선택할 때 참고 자료로 사용한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.usage-budget
    type: reads
    note: ""
  - to: concept.task
    type: reads
    note: ""
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

태스크를 실행하기 전에, 같은 프로젝트 내에서 동일한 모델 티어를 사용한 과거 기록의 평균 비용을 계산하여 예상 비용 추정치를 반환한다. 에이전트가 어떤 모델 티어를 선택할지 결정하는 데 참고 자료를 제공하는 것이 목적이다. 과거 기록이 없으면 `historical_avg_cost_usd` 가 null 이고 `samples_present` 가 false 로 반환된다.

## 요청 / 응답

**GET /tasks/:id/usage/preflight**
- 경로 파라미터: `id`(태스크 ID, 필수)
- 쿼리 파라미터: `tier`(모델 티어, 선택)
- 응답:
  - `task_id`: 조회 대상 태스크 ID
  - `tier`: 요청한 모델 티어 (요청 시 지정한 값 또는 기본값)
  - `historical_avg_cost_usd`: 과거 평균 비용 (USD). 참고 샘플이 없으면 null.
  - `samples_present`: 계산에 사용된 과거 기록이 존재하는지 여부 (true/false)

## 권한 / 제약

- 존재하지 않는 태스크 ID 를 지정하면 404 를 반환한다.
- 반환된 비용 추정치는 과거 평균에 기반한 휴리스틱이며, 실제 발생 비용을 보장하지 않는다.
- 이 엔드포인트는 읽기 전용이며 어떤 상태도 변경하지 않는다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 응답 구조는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `tier` 를 지정하지 않았을 때 기본값(예: 전체 티어 평균 vs 특정 기본 티어)이 무엇인지 미확인.
- 평균 계산에 사용하는 샘플 범위(예: 최근 N 건, 전체 기간)가 명세에 구체화되어 있지 않음.
- 태스크 유형이나 복잡도를 추가 변수로 고려하는지, 단순 티어별 평균만 사용하는지 미확인.
