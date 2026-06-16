---
id: endpoint.tasks-subtree
kind: Endpoint
title: "GET /tasks/:id/subtree"
definition: 지정한 태스크(루트)와 그 아래 모든 하위 태스크를 하나의 응답으로 반환한다. 대시보드에서 태스크 전체 트리를 한 번에 표시할 때 사용한다.
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

특정 태스크(`:id`)를 루트로 삼아, 해당 태스크 자신과 그 아래 모든 하위 태스크를 한 번에 반환한다. descendants 엔드포인트와 달리 루트 태스크 자체도 응답에 포함된다. 이 엔드포인트 하나로 특정 태스크를 중심으로 한 전체 작업 계층 구조를 파악할 수 있다.

## 요청 / 응답

**요청**
- 메서드: GET
- 경로 파라미터: `id` — 서브트리의 루트가 될 태스크의 ID
- 쿼리 파라미터: 없음
- 요청 본문: 없음

**응답**
- 성공(200): 루트 태스크와 모든 하위 태스크를 포함하는 구조체. 트리 형태(중첩 구조) 또는 평탄화된 배열 형태로 반환될 수 있으며, 루트 태스크는 반드시 포함된다.
- 태스크 없음(404): 지정한 `id`에 해당하는 태스크가 존재하지 않을 때 반환된다.
- 하위 태스크 없는 경우(200): 루트 태스크만 포함된 응답을 반환한다.

## 권한 / 제약

- 특별한 인증 요건 없음(데몬 로컬 API 기준).
- 루트 태스크 자신이 반드시 응답에 포함된다는 점이 descendants 엔드포인트와 다른 핵심 차이다.
- 주로 대시보드 UI에서 태스크 트리를 그릴 때 단일 API 호출로 필요한 모든 데이터를 가져오기 위해 사용된다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/run.rs`
- ancestors, descendants 엔드포인트와 함께 태스크 계층 탐색 API 군을 구성한다. subtree 는 루트를 포함한 전체 하향 범위를 한 번에 제공하는 편의 엔드포인트다.

## 미확정 (OPEN)

- 응답 형태가 중첩 트리 구조인지 평탄화된 배열인지 구현 확인 전까지 미검증 상태다.
- 대규모 트리(수백 개 이상의 하위 태스크)에 대한 페이지네이션 또는 깊이 제한 존재 여부 미확인.
- 루트 태스크의 위치(응답 배열 첫 번째 vs 별도 필드 vs 중첩 구조 최상단)가 미확인이다.
