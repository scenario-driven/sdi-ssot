---
id: domain.plugin-runtime
kind: Domain
title: 플러그인 런타임 (Plugin Runtime)
purpose: "Claude Code 플러그인 셸, MCP 서버, 훅 시스템이 결합해 SDI 기능을 Claude Code 세션에서 사용 가능하게 하고, 메인 세션의 실행 게이트(D21 위임·활성 태스크·D29 클레임)를 PreToolUse 훅으로 강제한다."
definition: "plugin/ 디렉터리의 플러그인 셸, MCP stdio 서버(sdi mcp), PreToolUse·PostToolUse·UserPromptSubmit 훅을 담당하는 경계. 두 바이너리(sdi CLI + sdid 데몬)가 Claude Code 바깥에서 실행되고, 플러그인은 훅·MCP·슬래시 명령·에이전트 정의만 담은 얇은 셸이다. sdid가 HTTP API + SSE + 유닉스 소켓으로 상태를 제공한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
  - persona.solo-builder
relatesTo:
  - to: domain.knowledge-rag
    type: relates-to
    note: MCP search_knowledge 도구가 이 도메인을 통해 에이전트에 노출된다.
  - to: domain.agent-coordination
    type: relates-to
    note: MCP agent-note 도구가 AgentNote 블랙보드 접근을 제공한다.
  - to: domain.project-management
    type: relates-to
    note: 훅이 현재 cwd로 프로젝트를 조회해 올바른 게이트를 적용한다.
  - to: domain.autonomy
    type: relates-to
    note: PreToolUse 훅이 AutonomyPolicy를 조회해 결정 적용 게이트를 실행한다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: D26 패턴 integrity gate가 PreToolUse 훅에서 active pattern의 shape를 검증한다.
  - to: concept.run
    type: relates-to
    note: 각 에이전트 실행(AgentRun)이 이 런타임에서 추적된다.
governedBy: []
realizedBy: []
impacts:
  - domain.autonomy
  - domain.collaboration-patterns
  - domain.project-management
implementedIn:
  - "sdi-plugin/plugin/.claude-plugin/plugin.json"
  - "sdi-plugin/plugin/.mcp.json"
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/crates/cli/"
  - "sdi-plugin/crates/daemon/"
  - "sdi-plugin/crates/mcp/"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI가 Claude Code 안에서 실제로 작동하게 하는 기술 인프라를 담당한다. SDI의 모든 정책·엔티티·협상은 이 도메인이 제공하는 훅·MCP·슬래시 명령 인터페이스를 통해 에이전트와 연결된다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **두 바이너리 구조**: 
  - `sdi` (CLI): 모든 작업 관리 명령 + MCP stdio 서버 (`sdi mcp`). 에이전트와 빌더가 직접 호출하는 인터페이스.
  - `sdid` (데몬): 상태를 소유하는 장기 실행 프로세스. SQLite를 단일 writer로 소유하고 HTTP API·SSE·유닉스 소켓을 제공. sdi가 데몬에 요청해 상태를 변경한다.

- **플러그인 셸**: `plugin/` 디렉터리가 Claude Code 플러그인의 실체다. 훅 등록·MCP 연결·슬래시 명령 정의·에이전트 정의·대시보드 SPA를 담는다. 바이너리나 비즈니스 로직이 없는 얇은 셸이다.

- **PreToolUse 훅 게이트**: Claude Code가 도구를 호출하기 전에 SDI의 훅이 먼저 실행된다. 주요 검사:
  - **D21 위임 게이트**: `hookInput.agent_id` 부재(= 메인 세션의 직접 실행) + 변경 도구 → 차단.
  - **활성 태스크 게이트**: 활성 태스크 없이 변경 도구 사용 → 차단.
  - **D26 패턴 shape 게이트**: active pattern의 종류별 무결성 조건 검사.
  - **D29 자원 클레임 게이트**: 대상 파일이 다른 시나리오의 클레임과 겹치면 차단.

- **슬래시 명령**: `/scenario`, `/round`, `/plan`, `/req`, `/decide`, `/consensus`, `/autonomy`, `/agent-note`, `/pattern`, `/sdi-status`. `/goal`은 Claude Code 내장으로 SDI가 가로채지 않는다.

- **XDG 경로 불변**: 사용자 데이터는 절대로 `~/.claude/plugins/` 아래에 저장되지 않는다. SQLite는 `~/.local/share/sdi/`, 소켓은 `~/.cache/sdi/`, 설정은 `~/.config/sdi/`, 로그는 `~/.local/state/sdi/`.

- **대시보드 SPA**: `plugin/web/`의 React SPA를 sdid가 tower-http ServeDir로 직접 서빙. 별도 웹 서버 불필요.

포함되지 않는 것: 비즈니스 로직(각 도메인), SQLite 스키마(domain.governance-audit이 감사 로그, db 크레이트가 스토리지).

## 기능

- Claude Code 플러그인으로 SDI를 등록하고 실행 환경을 설정한다.
- PreToolUse 훅으로 D21·활성태스크·D26·D29 게이트를 실행한다.
- MCP stdio 서버로 에이전트에 SDI 도구를 노출한다.
- 슬래시 명령으로 빌더에게 SDI 조작 인터페이스를 제공한다.
- 대시보드 SPA를 sdid에서 직접 서빙한다.
- sdid 시작 시 XDG 경로 불변을 검증한다.

## 시스템 흐름

빌더가 `/plugin install sdi`로 플러그인을 설치하면 훅과 MCP가 Claude Code에 등록된다. sdid를 시작하면 SQLite를 초기화하고 HTTP 서버를 열며 대시보드를 서빙한다. 이후 Claude Code 세션에서 에이전트가 도구를 호출할 때마다 PreToolUse 훅이 실행되어 게이트를 통과시키거나 차단한다. 슬래시 명령은 sdi CLI를 호출해 데몬에 요청을 보낸다.

## 다른 도메인과의 관계

플러그인 런타임은 SDI의 모든 도메인 기능을 Claude Code 인터페이스로 연결하는 접착제다. 도메인이 "무엇을 할 것인가"를 정의하면, 플러그인 런타임이 "어떻게 에이전트·빌더와 연결할 것인가"를 처리한다.

## 미확정 (OPEN)
- [ ] OPEN: PostToolUse 훅이 파일 변경을 활성 태스크에 자동 기록하는 구현이 현재 hooks.json에 포함되어 있는지 확인 필요.
