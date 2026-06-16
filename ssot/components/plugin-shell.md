---
id: component.plugin-shell
kind: SystemComponent
title: SDI 플러그인 셸 (Claude Code 플러그인 컨테이너)
purpose: Claude Code가 SDI를 플러그인으로 인식하고 설치·활성화할 수 있도록 구성 파일, 스킬 정의, MCP 설정, 에이전트 명세를 한 디렉토리에 묶는 플러그인 컨테이너다. SDI의 물리적 본체인 Rust 바이너리들이 이 셸의 하위 구조를 통해 Claude Code 생태계에 통합된다.
realizedBy:
  - domain.plugin-runtime
  - domain.agent-coordination
implementedIn:
  - sdi-plugin/plugin
dependsOn:
  - component.daemon-supervisor
  - component.hooks-adapter
  - component.web-dashboard
  - component.mcp
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: component.hooks-adapter
    type: contains
    note: plugin/adapters/ 와 plugin/hooks/ 가 셸 안에 있다
  - to: component.daemon-supervisor
    type: contains
    note: plugin/daemon/ 과 plugin/bin/ 가 셸 안에 있다
  - to: component.web-dashboard
    type: contains
    note: plugin/web/ SPA가 셸 안에 있다
  - to: component.mcp
    type: contains
    note: .mcp.json 이 셸에 있고 sdi mcp를 Claude Code에 등록한다
  - to: domain.plugin-runtime
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

플러그인 셸(`plugin/`)은 Rust 워크스페이스와 별개로 보이지만, 동일 저장소 안에 공존하는 Claude Code 플러그인 구성 레이어다. 이 디렉토리가 없으면 SDI는 독립 CLI 도구로는 동작하지만 Claude Code 에이전트 환경에서 자동 연동되지 않는다.

셸이 담당하는 것은 네 가지다. 첫째, **플러그인 manifest**(`plugin.json`)로 Claude Code가 이 플러그인의 이름·버전·스킬 목록을 파악한다. 둘째, **MCP 설정**(`.mcp.json`)으로 `sdi mcp` 프로세스를 MCP stdio 서버로 Claude Code에 자동 등록한다. 셋째, **스킬 파일**(`plugin/skills/`)로 `/sdi-overview`, `/sdi-scenario`, `/sdi-round`, `/sdi-evidence` 등의 슬래시 커맨드를 정의한다. 넷째, **에이전트 명세**(`plugin/agents/`)로 11개 서브에이전트의 역할·도구·프롬프트를 정의한다.

셸은 코드 실행 로직을 직접 포함하지 않는다. 실행은 Rust 바이너리(`sdi`, `sdid`)와 훅 어댑터 스크립트(`adapters/`)가 담당하며, 셸은 이것들을 Claude Code에 연결하는 선언적 구성 허브다.

## 경계와 의존

플러그인 셸은 그 안에 포함된 하위 컴포넌트들(훅 어댑터, 데몬 수퍼바이저, 웹 대시보드, MCP)에 의존한다. 역방향 의존은 없다. Claude Code 런타임이 이 셸을 설치하고 각 설정을 읽어 실행 환경을 구성한다.

## 통신 패턴

플러그인 셸 자체는 런타임 통신이 없다. 선언적 파일들(JSON, Markdown)이 Claude Code 런타임에 의해 읽히고, 그 지시에 따라 훅 스크립트 실행, MCP 서버 기동, 스킬 로드가 이뤄진다.

## 하위 서브패키지 (책임 단위)

- **plugin.json**: 플러그인 메타데이터와 스킬 목록 선언.
- **.mcp.json**: `sdi mcp` MCP 서버 등록 설정.
- **plugin/skills/**: 4개 스킬 정의 파일(sdi-overview, sdi-scenario, sdi-round, sdi-evidence).
- **plugin/agents/**: 11개 서브에이전트 명세 파일.
- **plugin/commands/**: 슬래시 커맨드 구현 파일(선택적).

## 미확정 (OPEN)

- [ ] OPEN: 플러그인 설치 방식(dist 브랜치 배포 vs GitHub Releases 바이너리 fetch)이 결정되지 않았다. 현재 배포 전략은 미정으로 CLAUDE.md에 기록되어 있다.
