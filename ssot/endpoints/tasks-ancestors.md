---
id: endpoint.tasks-ancestors
kind: Endpoint
title: "GET /tasks/:id/ancestors"
definition: 주어진 태스크의 모든 상위(부모) 태스크를 계층 순서대로 반환한다. 태스크 분해 계보를 역방향으로 추적할 때 사용한다.
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

특정 태스크(`:id`)에서 루트 태스크까지 이어지는 부모 태스크 체인 전체를 반환한다. 태스크가 계층적으로 분해된 경우, 그 분해 경로를 위쪽 방향으로 따라가며 중간 조상을 모두 수집한다. 루트 태스크(부모가 없는 태스크)는 이 엔드포인트를 호출해도 빈 배열을 반환한다.

## 요청 / 응답

**요청**
- 메서드: GET
- 경로 파라미터: `id` — 조상을 조회할 대상 태스크의 ID
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**응답**
- 성공(200): 조상 태스크 배열. 가장 가까운 부모부터 루트 순서로 정렬된다. 각 항목은 태스크 ID, 제목, 상태 등 태스크 요약 필드를 포함한다.
- 태스크 없음(404): 지정한 `id`에 해당하는 태스크가 존재하지 않을 때 반환된다.

## 권한 / 제약

- 특별한 인증 요건 없음(데몬 로컬 API 기준).
- 조상이 없는 태스크(루트 태스크)에 대해서는 빈 배열로 정상 응답한다. 오류가 아니다.
- 반환 순서는 가장 가까운 부모 → 루트 방향이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- 이 엔드포인트는 태스크 분해 트리의 역방향 탐색을 지원하기 위해 추가된 계층 조회 API 군의 일원이다.

## 미확정 (OPEN)

- 응답 필드의 정확한 스키마(태스크 요약에 포함되는 필드 목록)가 구현 코드 확인 전까지 미검증 상태다.
- 삭제된 태스크(cancelled/done 이후 정리된 태스크)가 조상 체인에 포함될 경우 처리 방식이 명확하지 않다.
- 깊이 제한(최대 조상 깊이) 존재 여부 미확인.
