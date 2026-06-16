---
id: endpoint.plans-export
kind: Endpoint
title: "GET /plans/export"
definition: 프로젝트에 속한 모든 계획 범위 데이터를 하나의 JSON 번들로 내보내는 엔드포인트. 백업 및 다른 프로젝트로의 마이그레이션에 사용한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.plan
    type: reads
    note: ""
  - to: concept.scenario
    type: reads
    note: ""
  - to: concept.requirement
    type: reads
    note: ""
  - to: concept.decision
    type: reads
    note: ""
  - to: concept.round
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

특정 프로젝트에 속한 모든 계획 범위 데이터를 하나의 JSON 번들로 내보낸다. 번들에는 프로젝트 기본 정보와, 각 계획별로 계획 자체·요구사항·결정·시나리오·라운드·태스크가 모두 포함된다. 이 번들은 `/plans/import` 엔드포인트로 복원하거나 다른 프로젝트로 마이그레이션하는 데 사용한다.

## 요청 / 응답

**GET /plans/export**
- 쿼리 파라미터: `project_id`(필수)
- 응답: JSON 번들 객체
  - `project`: 프로젝트 기본 정보
  - `plans`: 계획 배열. 각 계획은 다음을 포함:
    - `plan`: 계획 기본 정보
    - `requirements`: 요구사항 배열
    - `decisions`: 결정 배열
    - `scenarios`: 시나리오 배열
    - `rounds`: 라운드 배열
    - `tasks`: 태스크 배열

## 권한 / 제약

- `project_id` 없이 호출하면 400 오류를 반환한다.
- 이 엔드포인트는 읽기 전용이며 어떤 상태도 변경하지 않는다.
- 대용량 프로젝트의 경우 응답 크기가 매우 커질 수 있으며, 스트리밍 지원 여부는 현재 명세상 불명확하다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 번들 구조는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- 번들의 MIME 타입이 `application/json` 인지, 파일 다운로드를 유도하는 `application/octet-stream` 인지 미확인.
- 특정 계획만 선택적으로 내보내는 필터(예: `plan_id`) 쿼리 파라미터 지원 여부 미확인.
- 번들에 첨부 파일, 에이전트 노트, 활동 이벤트가 포함되는지 미확인.
