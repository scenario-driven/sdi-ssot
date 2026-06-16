---
id: component.cli
kind: SystemComponent
title: sdi-cli / sdi (Rust CLI 바이너리)
purpose: 코딩 에이전트와 사람이 SDI의 모든 기능을 명령줄에서 사용하는 단일 진입점이다. 플랜·시나리오·라운드·결정·패턴 전체의 생명주기 명령을 제공하고, 같은 바이너리 안에 MCP stdio 서버를 내장해 Claude Code가 `sdi mcp` 서브커맨드로 컨텍스트를 가져올 수 있도록 한다.
realizedBy:
  - domain.scenario-management
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - domain.collaboration-patterns
  - domain.agent-coordination
  - domain.plugin-runtime
implementedIn:
  - sdi-plugin/crates/cli
dependsOn:
  - component.core
  - component.db
  - component.mcp
  - component.daemon
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: component.core
    type: depends-on
    note: 도메인 타입을 직접 참조한다
  - to: component.db
    type: depends-on
    note: 일부 읽기 전용 경로에서 sdi-db를 통해 로컬 접근 가능
  - to: component.mcp
    type: contains
    note: sdi mcp 서브커맨드는 sdi-mcp 크레이트를 stdio 서버로 기동한다
  - to: component.daemon
    type: depends-on
    note: 모든 상태 변경 및 대부분의 조회는 데몬 HTTP API를 통한다
  - to: domain.plugin-runtime
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

`sdi` 바이너리(sdi-cli 크레이트에서 빌드)는 SDI의 명령줄 진입점이다. 에이전트가 `sdi plan create`, `sdi scenario create`, `sdi round start`, `sdi decision create` 등을 호출해 SDI 워크플로우를 구동한다. 사람도 같은 명령으로 시스템을 조작하고 관찰한다.

제공하는 명령 영역은 다음과 같다. 프로젝트 등록과 관리, 플랜 작성과 승인(D8 게이트 통과 후 활성화), 요구사항·결정·시나리오의 CRUD와 상태 전이, 라운드 생성·활성화·결과 기록, 태스크 분해와 증거 기록, 협업 패턴 등록과 전환, 자율 정책 조회와 조정, bypass 무장(단일 도구 호출 한 번만 게이트 우회), doctor(XDG 경로 위반·데몬 상태 진단), 셸 자동완성 생성.

`sdi mcp` 서브커맨드를 실행하면 같은 바이너리가 MCP stdio 서버로 전환된다. 이 모드에서 Claude Code는 read-only 도구를 통해 플랜·시나리오·결정·패턴 등의 컨텍스트를 가져온다.

## 경계와 의존

CLI는 상태를 직접 보관하지 않는다. 모든 쓰기와 대부분의 읽기는 로컬 데몬(`component.daemon`)의 HTTP API를 경유한다. sdi-core를 직접 의존해 도메인 타입을 공유받으며, sdi-mcp를 포함해 `sdi mcp` 서브커맨드를 호스팅한다. sdi-db는 일부 로컬 진단 경로에서 직접 접근하지만 일반 흐름에서는 데몬을 거친다.

단일 정적 바이너리로 GitHub Releases에서 다운로드되어 설치된다. Rust 툴체인이 사용자 환경에 없어도 동작한다.

## 통신 패턴

명령 실행 시 데몬의 HTTP REST API를 호출한다(reqwest HTTP 클라이언트). 데몬이 내려가 있으면 명령에 따라 오류를 반환하거나 데몬 기동을 시도한다. `sdi mcp` 모드에서는 Claude Code와 stdio로 말하고 내부적으로 데몬 포트를 자동 발견해 HTTP 요청을 보낸다.

## 하위 서브패키지 (책임 단위)

- **도메인 명령군**: plan, requirement, scenario, decision, round, task, pattern, policy 각 엔티티의 create/view/list/update/delete/transition 명령.
- **운영 명령군**: daemon(start/stop/status), doctor, bypass(arm/disarm), completions.
- **MCP 호스트**: `sdi mcp` 서브커맨드 — sdi-mcp 크레이트를 stdio 서버로 기동.

## 미확정 (OPEN)

- [ ] OPEN: 데몬 미기동 시 CLI가 자동 기동을 시도하는 조건과 실패 처리 경계는 daemon 노드 및 실제 코드에서 확인 필요.
