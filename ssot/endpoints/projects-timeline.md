---
id: endpoint.projects-timeline
kind: Endpoint
title: "GET /projects/:project_id/timeline"
definition: 특정 프로젝트의 최근 활동 이벤트를 타임라인 피드 형태로 반환하는 엔드포인트. /activity 목록 조회를 특정 프로젝트로 스코핑하고 기본 반환 건수를 높인 별칭 경로다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/collab.rs
relatesTo:
  - to: concept.project
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

특정 프로젝트의 활동 이벤트를 시간 순서로 나열한 타임라인 피드를 반환한다. 내부적으로는 `/activity?project_id=:project_id` 조회와 동일하지만, 프로젝트 ID 를 경로 파라미터로 받고 기본 반환 건수가 200으로 더 높다. 대시보드의 타임라인 뷰에서 사용한다.

## 요청 / 응답

**GET /projects/:project_id/timeline**
- 경로 파라미터: `project_id`(필수)
- 쿼리 파라미터: `limit`(반환 최대 건수, 기본값 200)
- 응답: 활동 이벤트 배열, 최신 이벤트 우선 정렬. 각 이벤트는 id, kind, summary, actor, entity_id, payload, created_at 필드를 포함한다.

## 권한 / 제약

- 존재하지 않는 project_id 를 지정하면 빈 배열을 반환하는지, 404 를 반환하는지 현재 명세상 불명확하다.
- `/activity` 엔드포인트와 동일한 데이터를 다른 경로로 제공하므로, 두 경로 중 하나로 일관된 소비 패턴을 유지하는 것이 권장된다.

## provenance

collab.rs 라우터에서 구현된다는 정보와 `/activity` 의 프로젝트 스코핑 별칭이라는 정보는 라우터 파일 목록 및 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `limit` 의 상한값이 별도로 제한되는지 미확인.
- 페이지네이션(커서 기반 또는 오프셋 기반) 지원 여부 미확인.
- `/activity` 와 완전히 동일한 데이터 소스를 사용하는지, 별도 뷰 또는 집계 로직이 존재하는지 미확인.
