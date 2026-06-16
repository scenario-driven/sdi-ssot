---
id: endpoint.tasks-descendants
kind: Endpoint
title: "GET /tasks/:id/descendants"
definition: 주어진 태스크 아래에 있는 모든 하위 태스크(자식, 손자, 그 이하)를 반환한다. 부모 태스크에서 분해된 모든 작업을 목록으로 나열할 때 사용한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.task
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 태스크(`:id`)를 기준으로, 그 태스크에서 파생된 모든 하위 태스크를 재귀적으로 수집해 반환한다. 직접적인 자식 태스크뿐 아니라 자식의 자식, 그 이하 깊이까지 포함한다. 태스크가 얼마나 세분화되었는지 전체 범위를 파악하거나, 분해된 작업의 완료 상태를 집계할 때 활용된다. 대상 태스크 자체는 응답에 포함되지 않는다.

## 요청 / 응답

**요청**
- 메서드: GET
- 경로 파라미터: `id` — 하위 태스크를 조회할 기준 태스크의 ID
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**응답**
- 성공(200): 하위 태스크 배열. 응답에 포함되는 순서(BFS/DFS 순서 등)는 구현에 따라 다를 수 있다. 각 항목은 태스크 ID, 제목, 상태 등 태스크 요약 필드를 포함한다.
- 태스크 없음(404): 지정한 `id`에 해당하는 태스크가 존재하지 않을 때 반환된다.
- 하위 태스크가 없는 경우(200): 빈 배열로 정상 응답한다.

## 권한 / 제약

- 특별한 인증 요건 없음(데몬 로컬 API 기준).
- 하위 태스크가 없는 리프 태스크에 대해서는 빈 배열로 정상 응답한다. 오류가 아니다.
- 대상 태스크(`:id`) 자체는 반환 목록에 포함되지 않는다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- ancestors, subtree 엔드포인트와 함께 태스크 계층 탐색 API 군을 구성한다. descendants 는 하향 방향, ancestors 는 상향 방향, subtree 는 루트 포함 전체를 대상으로 한다.

## 미확정 (OPEN)

- 응답의 정렬 순서(BFS vs DFS, 생성 시각 기준 등)가 구현 확인 전까지 미검증 상태다.
- 깊이 제한(최대 몇 단계까지 내려가는지) 존재 여부 미확인.
- 상태 필터(예: 완료된 하위 태스크 제외 옵션) 지원 여부 미확인.
