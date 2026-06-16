---
id: endpoint.knowledge-search
kind: Endpoint
title: "GET /knowledge/search"
definition: 지식 항목에 대한 전문 검색(full-text search)을 수행한다. SQLite FTS5 엔진을 사용하며, MCP 서버가 LLM에 컨텍스트를 공급할 때 이 엔드포인트를 호출한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/knowledge.rs"
relatesTo:
  - to: concept.knowledge-entry
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

지식 항목의 제목과 본문을 대상으로 전문 검색(full-text search)을 수행하는 엔드포인트다. SQLite의 FTS5 모듈을 검색 엔진으로 사용한다. 검색어와 관련도가 높은 항목을 최대 `limit`개 반환한다.

이 엔드포인트의 주요 소비자는 MCP 서버다. MCP 서버는 LLM 에이전트가 작업 중일 때 이 엔드포인트를 호출해 관련 지식 항목을 찾아내고, 그 내용을 LLM의 컨텍스트로 주입한다. 이를 통해 LLM이 프로젝트별로 축적된 지식 기반 위에서 동작할 수 있게 된다.

## 요청 / 응답

**쿼리 파라미터**

- `project_id` (필수) — 검색 대상을 한정할 프로젝트 식별자.
- `q` (필수) — 검색어. 비어있는 문자열을 전달하면 400 오류를 반환한다.
- `scope` (선택, 기본값 `"rag"`) — 검색할 항목의 scope. 생략하면 `rag` scope 항목만 검색한다.
- `limit` (선택, 기본값 20) — 반환할 최대 결과 수.

**응답**

검색 결과 항목 배열을 반환한다. 각 항목에는 id, title, body, scope, kind, tags, source_ref 등이 포함된다. 관련도 점수(score)가 함께 반환될 수 있다.

**오류**

- `q`가 빈 문자열인 경우 → 400.
- `project_id`가 없는 경우 → 400.

## 권한 / 제약

- 검색은 읽기 전용이다. 데이터를 변경하지 않는다.
- 기본 scope가 `rag`이므로, `reference`나 `archive` scope 항목을 검색하려면 `scope` 파라미터를 명시해야 한다.
- FTS5 검색이므로 정확한 단어 일치 외에도 어근 일치, 접두어 검색 등 SQLite FTS5가 지원하는 쿼리 문법을 사용할 수 있다.
- `limit`의 최대 허용값이 별도로 제한되는지는 미확인이다.

## provenance

MCP 서버의 RAG(Retrieval-Augmented Generation) 파이프라인 핵심 엔드포인트다. `scope=rag`를 기본값으로 두어 LLM이 소비 가능한 항목만 기본 검색 대상으로 삼는다. `POST /knowledge`로 축적된 지식 항목이 이 엔드포인트를 통해 LLM에 공급된다. `sdi-plugin/crates/daemon/src/router/knowledge.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- FTS5 인덱스가 `title`과 `body` 외에 `tags`나 `kind` 필드도 포함하는지 미확인이다.
- 관련도 점수(relevance score)가 응답에 포함되는지 여부가 명시되지 않았다.
- `limit`의 상한값(최대 허용 limit)이 서버 측에서 제한되는지 미확인이다.
- 검색어가 여러 단어일 때 AND 검색인지 OR 검색인지 기본 동작이 명시되지 않았다.
- `GET /knowledge/search` 경로가 `GET /knowledge/:id`와 라우터 순서상 충돌하지 않도록 처리되어 있는지 확인이 필요하다.
