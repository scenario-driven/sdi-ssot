---
id: domain.knowledge-rag
kind: Domain
title: 지식 RAG (Knowledge RAG)
purpose: "에이전트가 새 세션을 시작할 때 이전 세션의 결정·시나리오·지식을 의미 검색으로 불러와 컨텍스트를 복원하고, 위키성 지식 항목을 버전 관리해 팀 내 암묵지를 명시지로 변환한다."
definition: "Knowledge 엔티티의 생성·갱신·검색(키워드 + 의미 하이브리드)을 담당하는 경계. 로컬 SQLite + FTS5 키워드 검색과 sqlite-vec 벡터 검색을 결합한 온디바이스 RAG다. 어떤 데이터도 외부 서버로 나가지 않는다. MCP read-only 도구(`search_knowledge`)로 에이전트가 접근한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.agent-coordination
    type: relates-to
    note: AgentNote(블랙보드)가 knowledge와 함께 에이전트 간 컨텍스트 공유의 두 채널 중 하나다.
  - to: domain.plugin-runtime
    type: relates-to
    note: MCP 서버가 이 도메인의 지식을 에이전트에 노출하는 접근 채널이다.
  - to: concept.knowledge-entry
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
governedBy: []
realizedBy: []
impacts:
  - domain.agent-coordination
implementedIn:
  - "sdi-plugin/crates/core/src/knowledge.rs"
  - "sdi-plugin/crates/daemon/src/router/knowledge.rs"
  - "sdi-plugin/crates/db/"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 "세션이 끊겨도 컨텍스트가 살아있다"는 SDI의 세션 간 연속성 보장을 담당한다. LLM의 컨텍스트 윈도우는 세션마다 초기화되지만, 로컬 SQLite에 저장된 지식·결정·시나리오는 남는다. 새 세션을 시작하는 에이전트가 MCP 도구로 관련 지식을 검색하면, 빌더가 다시 설명할 필요 없이 이전 맥락을 이어갈 수 있다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **Knowledge 엔티티**: 위키성 항목. 결정 이유, 설계 배경, 도메인 용어, 재사용 패턴 등을 자연어로 저장한다. 모든 지식은 플랜 또는 프로젝트에 귀속된다.

- **온디바이스 하이브리드 검색**: FTS5 키워드 검색과 sqlite-vec 벡터 의미 검색을 결합한다. 임베딩 모델이 로컬에서 실행되어 어떤 데이터도 외부로 나가지 않는다.

- **MCP read-only 접근**: 에이전트는 `search_knowledge` MCP 도구로만 지식을 읽는다. 지식 추가·갱신은 CLI나 데몬 API를 통해서만 가능하다.

- **세션 간 컨텍스트 복원 흐름**: 에이전트가 새 세션을 시작할 때 대시보드 컨텍스트(훅이 주입)와 함께 관련 지식을 검색해 "어제 어디까지 했는지"를 재구성한다.

포함되지 않는 것: AgentNote 블랙보드(domain.agent-coordination), 시나리오 자체의 GWT 내용(domain.scenario-management), 결정 ADR 내용(domain.decision-negotiation).

## 기능

- 지식 항목을 만들고 갱신한다.
- 키워드와 의미 쿼리를 결합해 지식을 검색한다.
- 플랜·프로젝트 스코프로 지식 검색 범위를 좁힌다.
- 온디바이스 임베딩 모델로 지식 항목을 인덱싱한다.

## 시스템 흐름

빌더나 에이전트가 중요한 설계 결정·배경 지식을 `sdi knowledge add`로 저장한다. 데몬이 온디바이스 임베딩 모델로 인덱싱한다. 새 세션의 에이전트가 `search_knowledge(query="...")` MCP 도구를 호출하면, 데몬이 FTS5 키워드 결과와 벡터 유사도 결과를 결합해 가장 관련성 높은 항목을 반환한다.

## 다른 도메인과의 관계

지식 RAG는 에이전트 조율(domain.agent-coordination)의 장기 기억 채널이다. AgentNote가 하나의 세션·라운드 안에서 에이전트 간 단기 공유 채널이라면, Knowledge는 세션을 넘나드는 장기 공유 채널이다. 플러그인 런타임(domain.plugin-runtime)이 MCP 인터페이스를 통해 이 도메인의 기능을 에이전트에 노출한다.

## 미확정 (OPEN)
- [ ] OPEN: sqlite-vec 벡터 검색이 현재 코드에서 실제로 활성화되어 있는지, 아니면 FTS5 키워드 검색만 활성화되어 있는지(README에서 "vector search deferred" 언급) 확인 필요.
