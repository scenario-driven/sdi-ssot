---
id: endpoint.scenarios-search
kind: Endpoint
title: "GET /scenarios/search"
definition: >
  SQLite FTS5 전문 검색을 사용하여 특정 플랜 내 시나리오를 검색하는 엔드포인트.
  plan_id와 검색어(q)가 필수이며, 결과는 관련도 순으로 정렬되어 반환된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario
    type: reads
  - to: concept.given-when-then
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

특정 플랜에 속하는 시나리오들을 자연어 키워드로 검색한다. SQLite FTS5(Full-Text Search) 엔진을 사용하여 시나리오의 제목, Given / When / Then 텍스트 등을 대상으로 전문 검색을 수행하며, 결과는 검색어와의 관련도(relevance) 순으로 정렬된다.

검색은 읽기 전용 동작이므로 어떠한 상태도 변경하지 않는다. 검색 범위는 `plan_id`로 지정한 플랜 내로 한정되므로, 플랜 간 검색 혼용을 방지한다.

## 요청 / 응답

- 메서드: GET
- 쿼리 파라미터:
  - `plan_id` (필수): 검색 대상 플랜의 식별자
  - `q` (필수, 비어 있어선 안 됨): 검색어 문자열
  - `limit` (선택, 기본값 20): 반환할 최대 결과 수
- 요청 본문: 없음
- 응답: 관련도 순으로 정렬된 시나리오 목록. 각 항목은 시나리오 전체 필드(또는 요약 필드) 포함
- `plan_id` 또는 `q`가 없거나 `q`가 빈 문자열이면 400 반환
- 결과가 없을 경우 빈 배열 반환

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 검색 범위는 지정한 `plan_id` 내로 한정된다. 플랜 전체를 가로지르는 전역 검색은 이 엔드포인트로는 불가능하다.
- retired 시나리오가 검색 결과에 포함되는지, 기본적으로 제외되는지 미확인이다.
- FTS5 인덱스 갱신 주기(시나리오 생성/수정 시 즉시 갱신 여부 또는 지연 인덱싱 여부) 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `GET /scenarios/search` 핸들러를 추론하여 작성하였다. SQLite FTS5 전문 검색 사용, `plan_id`·`q`·`limit` 파라미터 구조는 SDI 플러그인의 MCP 도구 `search_scenarios`의 시그니처와 일치하는 것으로 추론하였다.

## 미확정 (OPEN)

- retired 시나리오가 기본 검색 결과에 포함되는지, 아니면 별도 파라미터(예: `include_retired=true`)로만 포함할 수 있는지 확인 필요.
- FTS5 인덱스 대상 필드(제목, Given, When, Then 중 어느 것인지, 또는 전부인지) 미확인.
- `limit`의 최댓값 상한이 별도로 존재하는지 미확인.
- 검색 결과에 FTS5 relevance 점수(rank)가 응답 필드로 노출되는지 미확인.
- `q`에 SQLite FTS5 특수 문법(큰따옴표 구문, 접두어 검색 등)을 그대로 사용할 수 있는지, 아니면 단순 키워드만 허용하는지 미확인.
