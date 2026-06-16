---
id: endpoint.metrics
kind: Endpoint
title: "GET /metrics"
definition: Prometheus 텍스트 형식으로 SDI 데몬의 핵심 지표를 반환하는 엔드포인트. Prometheus 스크레이핑 또는 `sdi metrics` CLI 에서 소비한다.
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

SDI 데몬의 운영 지표를 Prometheus 텍스트 형식으로 반환한다. Prometheus 서버가 주기적으로 스크레이핑하거나, `sdi metrics` CLI 명령을 통해 수동으로 조회할 수 있다.

반환하는 지표 목록:
- `sdi_tasks_total`: 태스크 수를 상태(status)별로 레이블링하여 집계한 카운터
- `sdi_projects_total`: 전체 프로젝트 수
- `sdi_usage_cost_usd_total`: 누적 토큰 사용 비용(USD)
- `sdi_usage_records_total`: 누적 사용 기록 건수
- `sdi_usage_input_tokens_total`: 누적 입력 토큰 수
- `sdi_usage_output_tokens_total`: 누적 출력 토큰 수

## 요청 / 응답

**GET /metrics**
- 요청 파라미터: 없음
- 응답: `Content-Type: text/plain; version=0.0.4` (Prometheus 텍스트 형식)
  ```
  # HELP sdi_tasks_total Total number of tasks by status
  # TYPE sdi_tasks_total gauge
  sdi_tasks_total{status="todo"} 5
  sdi_tasks_total{status="in_progress"} 2
  ...
  ```

## 권한 / 제약

- 인증 없이 접근 가능한지, 로컬 전용으로 제한되는지 현재 명세상 불명확하다.
- 모든 프로젝트의 지표를 합산하여 반환하며, 프로젝트별 필터링은 지원하지 않는다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 지표 이름 목록은 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `sdi_tasks_total` 의 레이블이 상태(status)만인지, 프로젝트(project_id)도 포함하는지 미확인.
- 지표 유형이 카운터(counter)인지 게이지(gauge)인지 각 지표별로 확정되지 않음.
- 캐시 기간(scrape interval 과의 정합성)이 별도로 지정되어 있는지 미확인.
