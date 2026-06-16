---
id: endpoint.projects-handoff
kind: Endpoint
title: "GET /projects/:id/handoff"
definition: 새로운 Claude Code 세션이 시작될 때 프로젝트 컨텍스트를 빠르게 복원할 수 있도록, 프로젝트 정보·활성 계획·확인된 시나리오·진행 중인 태스크·최근 결정·최근 활동을 하나의 번들로 묶어 반환하는 엔드포인트.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.project
    type: reads
    note: ""
  - to: concept.handoff
    type: reads
    note: ""
  - to: concept.task
    type: reads
    note: ""
  - to: concept.scenario
    type: reads
    note: ""
  - to: concept.decision
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

새로운 Claude Code 세션을 시작할 때, 이전 세션에서 쌓인 모든 상태를 다시 읽지 않고도 빠르게 작업을 이어받을 수 있도록 설계된 세션 픽업 번들을 반환한다. 번들에는 프로젝트 기본 정보, 현재 활성 계획, 활성 계획에 속한 모든 확인(confirmed) 상태의 시나리오, 진행 중(in-flight) 및 대기 중(backlog) 태스크, 최근 결정 사항, 그리고 최근 50건의 활동 이벤트가 포함된다.

## 요청 / 응답

**GET /projects/:id/handoff**
- 경로 파라미터: `id`(프로젝트 ID, 필수)
- 응답: 세션 픽업 번들 객체
  - `project`: 프로젝트 기본 정보
  - `active_plan`: 현재 활성 계획 (없으면 null)
  - `confirmed_scenarios`: 활성 계획에 속한 confirmed 상태 시나리오 배열 (GWT 포함)
  - `tasks`: 진행 중(in_progress) 및 대기 중(todo/backlog) 태스크 배열
  - `recent_decisions`: 최근 결정 사항 배열
  - `recent_activity`: 최근 50건의 활동 이벤트 배열

## 권한 / 제약

- 존재하지 않는 프로젝트 ID 를 지정하면 404 를 반환한다.
- 활성 계획이 없는 경우 `active_plan` 은 null 이고 `confirmed_scenarios` 는 빈 배열이다.
- 이 엔드포인트는 읽기 전용이며 어떤 상태도 변경하지 않는다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 세션 픽업 번들의 구성 요소는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `recent_decisions` 에서 반환하는 결정 건수의 기본값 및 상한이 명세에 명시되어 있지 않음.
- 활성 계획이 없을 때 모든 계획 중 가장 최근 계획을 fallback 으로 사용하는지, 아니면 완전히 빈 상태로 반환하는지 미확인.
- `tasks` 에 cancelled 상태 태스크가 포함되는지 미확인.
