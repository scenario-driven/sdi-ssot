---
id: endpoint.knowledge-export
kind: Endpoint
title: "GET /knowledge/export"
definition: >
  프로젝트에 속한 지식 항목(knowledge entry) 전체를 JSON 번들로 내보낸다.
  백업이나 환경 간 이전(마이그레이션)에 사용한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/aggregate.rs"
relatesTo:
  - to: concept.knowledge-entry
    type: reads
    note: 프로젝트에 속한 모든 지식 항목을 읽어 내보낸다
  - to: concept.project
    type: reads
    note: project_id 쿼리 파라미터로 어느 프로젝트의 지식을 내보낼지 특정한다
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 프로젝트에 등록된 지식 항목을 한 번에 내보내는 엔드포인트다. 반환값은 `version: 1` 봉투(envelope) 안에 지식 항목 배열이 담긴 JSON 형식이다. 다른 프로젝트나 다른 배포 환경으로 RAG·참고 콘텐츠를 옮길 때 사용한다. 반대 방향 작업은 `POST /knowledge/import` 가 담당한다.

## 요청 / 응답

**요청**

- 메서드: `GET`
- 경로: `/knowledge/export`
- 쿼리 파라미터
  - `project_id` (필수): 내보낼 프로젝트의 식별자
  - `scope` (선택): 특정 범위(scope)로 지식 항목을 좁혀서 내보낸다. 생략하면 프로젝트 전체 항목을 반환한다.

**응답**

- 성공 시 HTTP 200
- 본문: `{ "version": 1, "knowledge": [ ...지식 항목 배열... ] }` 형태의 JSON
- 지식 항목 배열이 비어 있어도 봉투 구조는 항상 포함된다.

## 권한 / 제약

- `project_id` 쿼리 파라미터가 없으면 요청이 거부된다.
- 데몬이 실행 중이어야 응답을 받을 수 있다.
- 읽기 전용 작업이므로 데이터를 변경하지 않는다.

## provenance

- 라우트 출처: `sdi-plugin/crates/daemon/src/router/aggregate.rs`
- 반환 형식의 `version: 1` 봉투는 향후 스키마 변경 시 하위 호환성을 위해 존재한다.

## 미확정 (OPEN)

- `scope` 파라미터의 허용 값 목록이 명세에 정의되지 않았다. 코드 확인 필요.
- 내보낸 JSON 번들의 최대 크기 제한이 있는지 불명확하다.
- 인증·권한 검사 여부가 명시되지 않았다.
