---
id: endpoint.plans-usage
kind: Endpoint
title: "GET /plans/:id/usage"
definition: 특정 계획에 속한 모든 사용량 기록을 집계하여 총 비용·토큰 수 요약과 원시 기록 목록을 함께 반환하는 엔드포인트. 대시보드 계획 상세 화면과 `sdi usage` CLI 에서 소비한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.usage-budget
    type: reads
    note: ""
  - to: concept.plan
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

특정 계획에 연결된 모든 토큰 사용 기록을 집계하여 요약 통계(총 비용, 기록 건수, 총 입력·출력 토큰 수)와 원시 기록 목록을 함께 반환한다. 대시보드의 계획 상세 화면에서 비용 현황을 보여주거나 `sdi usage` CLI 로 계획별 예산 소비 현황을 조회할 때 사용한다.

## 요청 / 응답

**GET /plans/:id/usage**
- 경로 파라미터: `id`(계획 ID, 필수)
- 응답:
  - `summary`: 집계 요약 객체
    - `total_cost_usd`: 총 비용 (USD)
    - `records_count`: 기록 건수
    - `total_input_tokens`: 총 입력 토큰 수
    - `total_output_tokens`: 총 출력 토큰 수
  - `records`: 원시 사용 기록 배열 (각 기록은 모델, 토큰 수, 비용, 타임스탬프 등 포함)

## 권한 / 제약

- 존재하지 않는 계획 ID 를 지정하면 404 를 반환한다.
- 사용 기록이 없는 계획의 경우 `summary` 는 모두 0, `records` 는 빈 배열을 반환한다.
- 이 엔드포인트는 읽기 전용이다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 집계 구조는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `summary` 에 `cache_read`/`cache_write` 토큰 집계가 포함되는지 미확인.
- `records` 배열에 페이지네이션이 적용되는지 미확인(계획이 장기간 운영되면 기록 건수가 매우 많아질 수 있음).
- 태스크별 또는 라운드별 세부 집계(breakdowns)가 별도 필드로 제공되는지 미확인.
