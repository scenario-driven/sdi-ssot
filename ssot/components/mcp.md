---
id: component.mcp
kind: SystemComponent
title: sdi-mcp (Rust MCP stdio 서버 크레이트)
purpose: Claude Code가 SDI 컨텍스트를 read-only로 가져오는 MCP(Model Context Protocol) 서버다. sdi CLI 바이너리에 내장되어 `sdi mcp` 서브커맨드로 기동되며, 에이전트가 플랜·시나리오·결정·패턴 상태를 조회해 작업 컨텍스트를 자동으로 보강한다.
realizedBy:
  - domain.agent-coordination
  - domain.plugin-runtime
  - domain.knowledge-rag
implementedIn:
  - sdi-plugin/crates/mcp
dependsOn:
  - component.db
  - component.daemon
consumesApi: []
providesApi: []
integratesWith:
  - integration.mcp-protocol
relatesTo:
  - to: component.cli
    type: belongs-to
    note: sdi-mcp는 sdi-cli 바이너리 안에 서브커맨드로 내장되어 독립 바이너리가 없다
  - to: component.db
    type: depends-on
    note: 일부 조회는 sdi-db를 직접 사용할 수 있다
  - to: component.daemon
    type: depends-on
    note: 데몬 HTTP API를 통해 최신 상태를 읽는다
  - to: domain.agent-coordination
    type: backed-by
  - to: domain.plugin-runtime
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

sdi-mcp는 Claude Code와 SDI를 연결하는 MCP stdio 서버다. Claude Code가 `plugin/.mcp.json` 설정에 따라 `sdi mcp`를 서브프로세스로 띄우면, 이 크레이트가 stdio 프로토콜로 MCP 도구 목록을 노출한다.

MCP 도구는 모두 read-only(조회 전용)다. 에이전트가 현재 활성 플랜의 시나리오 목록, 특정 라운드의 결과, 진행 중인 결정의 상태, 활성 패턴의 형상 등을 MCP 도구 호출로 가져온다. 이 컨텍스트를 바탕으로 에이전트는 어떤 시나리오에 연결된 작업을 해야 하는지, 현재 어떤 결정이 협상 중인지, 어떤 패턴이 적용 중인지 파악한다.

플러그인 manifest(`.mcp.json`)가 이 서버를 Claude Code에 자동 등록하며, 에이전트 세션 시작 시 MCP 도구로 즉시 사용 가능해진다.

## 경계와 의존

sdi-mcp는 쓰기 도구를 노출하지 않는다. 모든 SDI 뮤테이션은 CLI 명령이나 훅 어댑터가 데몬 HTTP API를 호출하는 경로를 사용한다. MCP는 읽기 컨텍스트 보강 전용 채널이다.

데몬 HTTP API를 통해 상태를 읽고, 데몬이 내려가 있을 때를 위해 sdi-db에 직접 접근하는 경로도 가질 수 있다. 독립 실행 바이너리가 없으며 sdi-cli 바이너리 안에 라이브러리로 포함된다.

## 통신 패턴

Claude Code ↔ sdi-mcp는 stdio(표준입출력) JSON-RPC로 통신한다. sdi-mcp ↔ 데몬은 내부적으로 reqwest HTTP 클라이언트를 통해 통신한다. 데몬 포트는 XDG 캐시 경로에 기록된 port 파일에서 자동 발견한다.

## 하위 서브패키지 (책임 단위)

- **MCP 프로토콜 핸들러**: stdio JSON-RPC 파싱과 도구 목록 응답.
- **SDI 조회 도구군**: `get_plan_context`, `search_scenarios`, `get_recent_decisions`, `list_agent_notes`, `get_autonomy_state`, `start_round` 등 플러그인이 노출하는 read-mostly MCP 도구 구현.
- **데몬 연결**: 데몬 포트 자동 발견과 HTTP 요청 클라이언트.

## 미확정 (OPEN)

- [ ] OPEN: 데몬 미가동 시 sdi-db 직접 접근 여부와 그 경우 캐시 일관성 보장 방법 확인 필요.
