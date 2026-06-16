---
id: invariant.mcp-tools-readonly
kind: Invariant
title: MCP read 도구는 scope=rag 데이터만 반환한다
definition: "MCP 서버가 노출하는 읽기 전용 도구(search_knowledge, find_similar_tasks, get_recent_decisions, get_plan_context)는 scope='rag'인 knowledge entry만 필터링하여 반환한다. scope='reference'나 내부 관리 데이터가 MCP read 경로로 유출되지 않는다."
governs:
  - domain.knowledge-rag
  - domain.plugin-runtime
  - concept.knowledge-entry
  - concept.agent-note
implementedIn:
  - "sdi-plugin/crates/mcp/src/tools/read.rs"
  - "sdi-plugin/crates/daemon/src/router/knowledge.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 MCP 서버는 Claude Code LLM이 시나리오·계획·지식을 조회하는 인터페이스다. MCP read 도구(`search_knowledge`, `find_similar_tasks`, `get_recent_decisions`, `get_plan_context`)는 LLM이 현재 작업 컨텍스트를 이해하기 위한 RAG 조회 경로다.

이 조회는 `scope='rag'`로 분류된 항목만 반환해야 한다. `scope='reference'`는 내부 참조 데이터로 MCP를 통한 LLM 노출 대상이 아니다. 필터 없이 전체 knowledge를 반환하면 내부 관리 데이터, 시스템 설정, 민감한 메타데이터가 LLM 컨텍스트에 주입될 수 있다.

MCP 서버 자체는 read-only 도구와 write 도구를 모두 노출한다(`add_scenario`, `add_requirement`, `add_decision`, `update_task_evidence`, `start_round`). 이 불변식은 read 도구의 반환 범위만을 제약하는 것이다.

## 깨지면 무슨 일이 일어나나

`scope='reference'` 데이터가 MCP read 경로로 유출되면 LLM이 내부 관리 데이터를 시나리오 내용이나 요구사항으로 오인한다. 예를 들어 시스템 설정값이 `search_knowledge` 결과로 나오면 LLM이 그것을 현재 작업의 컨텍스트로 해석하여 잘못된 Task를 분해하거나, 잘못된 GWT 시나리오를 작성할 수 있다. 또한 민감한 내부 경로나 설정이 LLM 컨텍스트 윈도우에 포함되어 log나 외부 API 호출에 노출될 위험이 있다.

## 코드에서 어떻게 강제되나

MCP read 도구 핸들러(`crates/mcp/src/tools/read.rs`)는 knowledge 조회 SQL에 `WHERE scope = 'rag'` 필터를 적용한다. 이 필터는 핸들러 코드에 하드코딩되어 있으며 외부에서 override할 수 없다. HOOK_ENFORCEMENT.md #9 인수기준과 `read_tools::search_knowledge_never_leaks_reference_scope` 테스트로 검증된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(MCP read 도구 scope=rag 필터 정책의 결정 엔트리) 연결 필요
