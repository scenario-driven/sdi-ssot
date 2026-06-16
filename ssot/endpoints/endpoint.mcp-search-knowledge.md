---
id: endpoint.mcp-search-knowledge
kind: Endpoint
title: MCP search_knowledge — 지식 베이스 전문 검색
definition: LLM 에이전트가 프로젝트 RAG 지식 항목을 FTS5 전문 검색으로 찾는 MCP 읽기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/read.rs
relatesTo:
  - to: concept.knowledge-entry
    type: reads
  - to: domain.knowledge-rag
    type: belongs-to
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`search_knowledge` 는 SDI MCP 서버의 읽기 도구로, LLM 에이전트가 작업에 필요한 도메인 지식, 과거 결정 배경, 프로젝트 맥락 등을 지식 베이스에서 검색할 때 사용한다. 데몬의 `GET /knowledge?scope=rag&q=...` 엔드포인트로 위임되며, SQLite FTS5 엔진을 통해 전문(full-text) 검색을 수행한다.

이 도구가 검색하는 지식 항목은 `scope=rag` 로 등록된 항목만이다. RAG 스코프 항목은 에이전트가 작업 중 참고할 수 있도록 의도적으로 색인된 정보다. 위키 형태의 일반 문서(`scope=wiki`)나 결정 아카이브(`scope=decision`)는 이 도구로 접근할 수 없으며, 별도 조회 경로가 필요하다.

## 요청 / 응답

**입력이 의미하는 것**

검색어는 자연어 구문이나 키워드 형태 모두 가능하며, FTS5 MATCH 구문으로 변환되어 색인에서 매칭된다. 검색어가 어떤 언어로 작성되어도 색인된 내용과의 언어적 일치가 중요하다. 선택적으로 프로젝트 ID를 지정하면 해당 프로젝트의 지식 항목으로 검색 범위를 좁힐 수 있다.

**출력이 의미하는 것**

응답은 관련성 순으로 정렬된 지식 항목 목록이다. 각 항목은 제목, 요약 또는 본문 스니펫, 스코프(이 도구에서는 항상 `rag`), 관련성 근거가 되는 매칭 구문을 포함한다. 에이전트는 이 결과를 바탕으로 작업 컨텍스트를 보강하거나, 유사한 선례가 있는지 확인하거나, 결정에 필요한 배경 지식을 확보한다.

## 권한 / 제약

이 도구는 읽기 전용이며 지식 베이스를 변경하지 않는다.

`scope=rag` 고정이다. 이 도구로는 위키 전용으로 등록된 항목이나 결정 아카이브에 접근할 수 없다. 이는 의도된 설계로, 에이전트가 참조용으로 색인된 항목만 소비하고 전체 지식 베이스를 무분별하게 탐색하지 않도록 범위를 제한한다.

FTS5 MATCH 구문은 특수문자에 민감하다. 검색어에 따옴표, 괄호, 하이픈 등이 포함된 경우 예상치 못한 결과가 나올 수 있으며, MCP 레이어에서 특수문자를 이스케이프 처리하는지 여부는 미확인이다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/read.rs` 구현 분석에서 추론
- 위임 엔드포인트: `GET /knowledge?scope=rag&q=...` (데몬 REST API)
- 관련 설계 결정: RAG 스코프 분리 원칙, FTS5 검색 엔진 선택

## 미확정 (OPEN)

- 반환 결과의 최대 개수와 관련성 점수 임계값이 불명확
- 검색어 특수문자 이스케이프를 MCP 레이어가 처리하는지, 호출자가 처리해야 하는지 미확인
- `scope=wiki` 또는 `scope=decision` 항목을 이 도구나 별도 도구로 접근할 수 있는지 불명확
- 지식 항목 등록(write) 도구가 MCP 레이어에 노출되어 있는지 미확인 — 등록은 CLI 경로만 지원되는지 확인 필요
