---
id: endpoint.activity-stats
kind: Endpoint
title: "GET /activity/stats"
definition: 프로젝트의 활동 이벤트를 이벤트 종류(kind)별로 집계하여 건수를 반환하는 엔드포인트. 대시보드의 활동 분포 차트에서 소비한다.
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

특정 프로젝트 안에서 발생한 활동 이벤트를 이벤트 종류(kind) 기준으로 그룹화하여 각 종류별 건수를 반환한다. 대시보드가 프로젝트 활동 분포를 시각적 차트로 보여줄 때 이 엔드포인트의 데이터를 사용한다.

## 요청 / 응답

**GET /activity/stats**
- 쿼리 파라미터: `project_id`(필수)
- 응답: 이벤트 종류(kind)를 키로, 해당 종류의 이벤트 건수를 값으로 하는 객체. 예: `{ "task_completed": 12, "scenario_confirmed": 5, "round_started": 3 }`

## 권한 / 제약

- `project_id` 없이 호출하면 400 오류를 반환한다.
- 조회 대상 기간(날짜 범위) 필터가 지원되는지 현재 명세상 불명확하다.

## provenance

collab.rs 라우터에서 구현된다는 정보는 라우터 파일 목록 분석을 통해 추론되었다.

## 미확정 (OPEN)

- 집계 결과에 건수가 0인 이벤트 종류가 포함되는지, 아니면 1건 이상인 종류만 포함되는지 미확인.
- 날짜 범위 필터(예: `from`, `until`) 쿼리 파라미터 지원 여부 미확인.
- 반환 형태가 배열(예: `[{ kind, count }]`)인지 맵 객체인지 미확인.
