---
id: endpoint.tasks-brief
kind: Endpoint
title: "GET /tasks/:id/brief"
definition: 특정 태스크에 대한 위임 브리핑 번들을 반환하는 엔드포인트. 강한 모델이 브리핑을 작성하고 약한 모델이 그 브리핑을 따르는 구조를 지원하기 위해 설계되었다. `sdi task brief` CLI 에서 소비한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.task
    type: reads
    note: ""
  - to: concept.scenario
    type: reads
    note: "상위 시나리오의 완전한 GWT 가 인라인으로 포함됨"
  - to: concept.round-baseline
    type: reads
    note: ""
  - to: concept.task-evidence
    type: reads
    note: "증거 형식 문자열 및 보고서 스키마 템플릿이 포함됨"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

하나의 태스크에 대한 위임 브리핑 번들을 반환한다. 강한 모델(예: 고성능 LLM)이 이 엔드포인트로 브리핑을 조회하고, 실제 구현 작업은 약한 모델(예: 경량 LLM)이 그 브리핑을 따라 수행하는 역할 분리 구조를 지원한다. 번들에는 태스크 자체, 해당 라운드, 라운드 베이스라인, 상위 시나리오(Given/When/Then 전체 인라인 포함), 증거 형식 문자열, 보고서 스키마 템플릿, 그리고 금지 사항 목록(git commit, push, reset 불가 등)이 포함된다. `sdi task brief` CLI 명령에서 이 엔드포인트를 호출한다.

## 요청 / 응답

**GET /tasks/:id/brief**
- 경로 파라미터: `id`(태스크 ID, 필수)
- 응답:
  - `task`: 태스크 기본 정보
  - `round`: 해당 태스크가 속한 라운드
  - `baseline`: 라운드 베이스라인 (없으면 null)
  - `scenarios`: 상위 시나리오 배열 (각 시나리오의 Given/When/Then 전문 인라인 포함)
  - `evidence_format`: 증거 제출 형식 문자열
  - `report_schema`: 보고서 구조 템플릿
  - `prohibitions`: 금지 행위 목록 (예: git commit/push/reset 불가)

## 권한 / 제약

- 존재하지 않는 태스크 ID 를 지정하면 404 를 반환한다.
- 이 엔드포인트는 읽기 전용이며 어떤 상태도 변경하지 않는다.
- 금지 사항 목록은 약한 모델이 허용되지 않는 행위를 실수로 수행하지 않도록 안전망 역할을 한다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 번들 구성 요소는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `prohibitions` 목록이 코드 내 하드코딩인지, 설정 파일에서 읽어오는지 미확인.
- 태스크에 연결된 상위 시나리오가 없는 경우(orphan task) 의 처리 방식 미확인.
- `evidence_format` 및 `report_schema` 가 태스크 또는 계획 단위로 커스터마이징 가능한지 미확인.
