---
id: endpoint.knowledge
kind: Endpoint
title: "POST /knowledge, GET /knowledge"
definition: 지식 항목을 새로 생성하거나 프로젝트 단위로 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/knowledge.rs"
relatesTo:
  - to: concept.knowledge-entry
    type: mutates
  - to: concept.project
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

두 가지 동작을 담당하는 엔드포인트다.

`POST /knowledge`는 새로운 지식 항목(knowledge entry)을 생성한다. 지식 항목은 LLM이나 MCP 서버가 참고할 수 있는 정형화된 텍스트 블록으로, 프로젝트 단위로 관리된다. 항목이 어떤 용도로 노출될지는 `scope` 필드로 구분한다.

`GET /knowledge`는 특정 프로젝트에 속한 지식 항목 목록을 반환한다. 조회 시 `scope`로 필터링할 수 있다.

## 요청 / 응답

**POST /knowledge 요청 본문 (JSON)**

- `project_id` (필수) — 이 지식 항목이 속할 프로젝트 식별자.
- `scope` (선택, 기본값 `"rag"`) — 항목의 노출 범위. 가능한 값:
  - `rag` — MCP/LLM이 컨텍스트로 소비할 수 있는 항목.
  - `reference` — 참고 자료. LLM에 자동 노출되지 않는다.
  - `archive` — 보관용. 활성 사용에서 제외된 항목.
- `kind` (필수) — 항목의 분류 유형. 예: `decision`, `guideline`, `context` 등.
- `title` (필수) — 항목의 제목.
- `body` (필수) — 항목의 본문 내용.
- `tags` (선택) — 항목에 붙일 태그 목록.
- `source_ref` (선택) — 이 지식의 출처 참조(문서 URL, 커밋 해시, 파일 경로 등).

**POST 응답**

생성된 지식 항목의 전체 데이터를 반환한다. 생성된 `id`가 포함된다.

**GET /knowledge 쿼리 파라미터**

- `project_id` (필수) — 조회할 프로젝트 식별자.
- `scope` (선택) — 특정 scope의 항목만 반환. 생략하면 전체 scope를 반환한다.

**GET 응답**

조건에 맞는 지식 항목 배열을 반환한다.

## 권한 / 제약

- `GET /knowledge`에서 `project_id`는 필수다. 생략하면 요청이 거부된다.
- `scope`는 `rag`, `reference`, `archive` 중 하나여야 한다. 정의되지 않은 scope 값은 거부될 수 있다.
- `rag` scope 항목만 MCP 서버가 LLM 컨텍스트로 자동 주입한다. 다른 scope의 항목은 명시적 조회 없이는 LLM에 전달되지 않는다.

## provenance

MCP/LLM에 컨텍스트를 공급하는 RAG 파이프라인의 입력 계층으로 설계되었다. `scope=rag` 항목이 `GET /knowledge/search`와 함께 MCP 서버가 LLM에 공급하는 지식의 원천이 된다. `sdi-plugin/crates/daemon/src/router/knowledge.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- `kind` 필드에 허용되는 값의 전체 목록이 열거형으로 제한되는지, 자유 문자열인지 미확인이다.
- `tags` 필드의 저장 방식(JSON 배열, 쉼표 구분 문자열 등)이 미확인이다.
- POST 성공 시 SSE 이벤트(`knowledge.created` 등)가 발행되는지 여부가 명시되지 않았다.
- 동일 프로젝트 내에서 `title`의 중복 허용 여부가 미확인이다.
