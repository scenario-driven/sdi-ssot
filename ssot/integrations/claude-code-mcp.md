---
id: integration.claude-code-mcp
kind: Integration
title: Claude Code MCP 서버 연동
purpose: LLM 코딩 에이전트가 SDI 데몬에 축적된 시나리오·지식·결정·태스크 컨텍스트를 자연어 쿼리로 끌어오고, 새 시나리오·요구사항·결정을 바로 적재할 수 있도록 — `sdi` CLI가 stdio MCP 서버를 노출해 Claude Code의 도구 호출 인터페이스와 연결한다.
definition: "`sdi mcp` 서브커맨드가 MCP stdio(JSON-RPC 2.0 뉴라인 구분) 서버를 기동한다. Claude Code는 `.mcp.json` 등록을 통해 이 서버를 자동으로 띄우고, 6개 읽기 도구(RAG 전용)와 5개 쓰기 도구(데몬 HTTP 중계)를 LLM 컨텍스트에서 직접 호출한다."
integratesWith: []
implementedIn:
  - sdi-plugin/crates/mcp/src/lib.rs
  - sdi-plugin/crates/mcp/src/server.rs
  - sdi-plugin/crates/mcp/src/tools/read.rs
  - sdi-plugin/crates/mcp/src/tools/write.rs
  - sdi-plugin/plugin/.mcp.json
impacts:
  - domain.knowledge-rag
  - domain.scenario-management
  - domain.decision-negotiation
  - domain.planning
  - domain.plugin-runtime
relatesTo:
  - to: domain.knowledge-rag
    type: realizes
    note: search_knowledge 도구가 scope=rag 엔트리만 노출하는 RAG 도메인의 외부 진입점
  - to: domain.scenario-management
    type: realizes
    note: search_scenarios·add_scenario 도구가 시나리오 관리 도메인을 MCP 표면으로 노출
  - to: domain.decision-negotiation
    type: realizes
    note: get_recent_decisions·add_decision 도구가 결정 협상 도메인과 LLM을 연결
  - to: domain.planning
    type: realizes
    note: get_plan_context·add_requirement·start_round가 계획 도메인을 LLM이 읽고 변경할 수 있게 한다
  - to: integration.claude-code-hooks
    type: relates-to
    note: 같은 플러그인 셸이 등록하는 두 접점 — 훅은 게이트·기록, MCP는 RAG 조회·엔티티 변경
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 LLM 코딩 에이전트인 Claude Code다. SDI는 `sdi mcp` 서브커맨드를 통해 MCP stdio 서버를 노출한다. Claude Code는 플러그인의 `.mcp.json` 등록을 읽어 이 서버를 `sdi mcp`로 기동하고, LLM이 작업 중에 도구 목록에서 SDI 도구를 직접 호출한다.

프로토콜은 MCP stdio 변형이다. 표준입력에서 뉴라인 구분 JSON-RPC 2.0 요청을 읽고, 표준출력으로 응답을 돌려보낸다. Content-Length 헤더 프레이밍은 사용하지 않는다.

노출 도구는 두 계층이다.

**읽기 도구(6종)** — RAG 전용이며 데이터를 변경하지 않는다:
- `search_knowledge`: 프로젝트 지식 엔트리 전문검색. FTS5 기반. `scope=rag` 엔트리만 반환한다(reference·archive는 LLM에게 보이지 않는다).
- `search_scenarios`: 플랜의 시나리오를 자연어 키워드로 검색한다.
- `get_plan_context`: 플랜 전체 컨텍스트(시나리오·요구사항·태스크 현황)를 가져온다.
- `get_recent_decisions`: 프로젝트의 최근 결정 목록을 반환한다.
- `get_autonomy_state`: 현재 AutonomyPolicy 상태를 조회한다.
- `list_agent_notes`: 에이전트가 남긴 노트 목록을 반환한다.

**쓰기 도구(5종)** — 데몬 HTTP API로 중계하는 변경 도구:
- `add_scenario`: 플랜에 Given/When/Then 시나리오를 추가한다. 데몬이 D5 GWT 비어있음 조건을 강제한다.
- `add_requirement`: 프로젝트에 요구사항(스냅샷)을 추가한다.
- `add_decision`: append-only ADR 형식의 결정을 등록한다.
- `update_task_evidence`: 태스크 완료 시 증거를 구조화된 형식으로 기록한다.
- `start_round`: 새 라운드를 시작한다.

서버는 기동 전에 데몬(sdid)이 실행 중임을 확인한다. 데몬이 없으면 시작하지 않는다.

## 구현 위치 (provenance)

MCP 서버의 진입점은 `crates/mcp/src/lib.rs`의 `run_stdio` 함수다. `crates/cli`의 `sdi mcp` 서브커맨드가 이 함수를 호출한다. 서버 루프(요청 수신·디스패치·응답 전송)는 `crates/mcp/src/server.rs`에 있고, 도구 정의는 `crates/mcp/src/tools/read.rs`(읽기)와 `crates/mcp/src/tools/write.rs`(쓰기)로 분리되어 있다.

Claude Code가 이 서버를 어떻게 띄우는지는 `plugin/.mcp.json`이 선언한다. 이 파일은 `sdi mcp` 명령을 stdio 타입으로 등록한다. 플러그인이 설치될 때 Claude Code가 이 파일을 읽어 자동으로 서버를 관리한다.

쓰기 도구들은 입력 인자를 데몬 HTTP 본문으로 변환해 POST 요청을 보낸다. 유효성 검사는 데몬에서만 이뤄지며(GWT 비어있음, 시나리오 미완비, 증거 필수 등) MCP 레이어에는 중복 검사를 두지 않는다.

## 불변식

- 읽기 도구는 `scope=rag` 엔트리만 반환해야 한다. `reference`·`archive` 스코프 엔트리가 LLM에 노출되면 안 된다. 이 제약은 쿼리 파라미터를 서버 측에서 강제하므로 LLM이 스코프를 다른 값으로 지정해도 무시된다.
- 쓰기 도구는 반드시 데몬을 통해서만 변경한다. MCP 레이어가 SQLite에 직접 접근하는 일은 없다.
- MCP 서버는 데몬이 먼저 실행 중이어야 동작한다. 서버 시작 시 포트 파일(`~/.cache/sdi/sdid.port`)을 읽어 데몬 주소를 확인한다.

## 영향 범위

이 연동이 끊기면 LLM이 세션 간 컨텍스트(시나리오·결정·지식)를 끌어오는 경로가 사라지고, MCP를 통한 엔티티 생성도 불가능해진다. 훅 연동(`integration.claude-code-hooks`)이 게이트와 기록을 담당하는 것과 달리, 이 연동은 LLM이 SDI 데이터를 직접 읽고 쓰는 인터페이스 경로 자체다.

## 미확정 (OPEN)
- [ ] OPEN: MCP 응답 크기 상한과 그 초과 시 "잘림" 안내 메시지의 정확한 정책이 코드 상에서 어디에 정의되는지 명시 필요.
- [ ] OPEN: 읽기 도구와 쓰기 도구가 동일한 서버 프로세스에서 노출되는 현재 구조에서, 쓰기 도구를 완전히 끌 수 있는 옵션(읽기 전용 모드)의 필요 여부 미결정.
