---
id: capability.search-knowledge
kind: Capability
title: 지식 검색 및 컨텍스트 조회
purpose: "에이전트가 세션 간 보존된 계획 맥락·시나리오·결정을 키워드나 의미 기반으로 찾아 현재 작업의 근거로 활용한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
  - persona.solo-builder
implementedIn:
  - sdi-plugin/plugin/skills/sdi-overview/SKILL.md
relatesTo:
  - { to: concept.knowledge-entry, type: reads, note: "rag 스코프 지식 엔트리를 키워드 검색으로 찾는다" }
  - { to: concept.scenario, type: reads, note: "활성 플랜의 시나리오를 키워드로 검색한다" }
  - { to: concept.decision, type: reads, note: "최근 의사결정 ADR 로그를 페이지네이션으로 조회한다" }
  - { to: concept.plan, type: reads, note: "플랜 + 시나리오 + 진행 중 태스크 + 결정의 복합 스냅샷을 한 번에 조회한다" }
impacts:
  - concept.knowledge-entry
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

새 세션을 시작했거나 이전에 내렸던 결정·시나리오 내용이 기억나지 않을 때, 데몬에 저장된 정보를 검색해 찾는다. MCP 도구 네 가지가 이 역할을 담당한다: 키워드로 지식 엔트리를 찾는 `search_knowledge`, 시나리오를 검색하는 `search_scenarios`, 플랜 전체 맥락을 한 번에 가져오는 `get_plan_context`, 최근 의사결정 이력을 보는 `get_recent_decisions`.

이 도구들은 모두 읽기 전용이고 rag 스코프 데이터만 반환한다. 현재 태스크 실행 상태나 데몬 헬스처럼 운영 상태는 슬래시 명령이나 HTTP API를 통해 직접 확인해야 한다. MCP가 기획·실행 전 컨텍스트 파악용이라면, `/sdi-status`는 실행 중 상태 확인용이다.

## 행위

- `search_knowledge` 도구로 키워드 기반 지식 엔트리 검색.
- `search_scenarios` 도구로 활성 플랜의 시나리오를 키워드로 검색.
- `get_plan_context` 도구로 플랜·시나리오·in-flight 태스크·결정을 복합 조회.
- `get_recent_decisions` 도구로 ADR 로그 최신 항목을 페이지네이션으로 조회.
- 웹 대시보드 WikiView에서 마크다운 지식 문서를 탐색할 수 있다.

## 시스템 흐름

에이전트(또는 사람)가 MCP 클라이언트를 통해 도구 호출 → `sdi mcp` 서버(stdio)가 수신 → 데몬 HTTP API를 거쳐 SQLite에서 rag 스코프 데이터 조회 → 결과 반환. 도구들은 reference나 archive 스코프 데이터를 LLM에게 노출하지 않는다.

## 어디에 구현되어 있나

MCP 서버는 `sdi mcp` 서브커맨드(`crates/mcp/`)로 실행된다. 4개 읽기 도구와 5개 쓰기 도구가 MCP 계약을 통해 노출된다. 웹 대시보드 WikiView(`plugin/web/src/views/WikiView.tsx`)가 마크다운 지식 문서 탐색 UI를 제공한다. `sdi-overview` 스킬(`plugin/skills/sdi-overview/SKILL.md`)이 도구 용도와 한계를 정의한다.

## 미확정 (OPEN)

- [ ] OPEN: `search_knowledge` 의 검색 방식(키워드 전문 검색, 벡터 유사도, 하이브리드)이 Clawket의 sqlite-vec 방식과 동일한지, SDI 데몬 구현이 다른지 확인 필요.
