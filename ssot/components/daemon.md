---
id: component.daemon
kind: SystemComponent
title: sdi-daemon / sdid (Rust 백그라운드 데몬)
purpose: SDI 모든 상태의 단일 소유자이자 진실 저장소다. axum HTTP 서버와 Unix 도메인 소켓으로 API를 제공하고, SQLite를 단독 소유하며, CLI·웹 대시보드·MCP 서버·훅 어댑터가 상태를 읽고 쓰는 중심 허브 역할을 한다.
realizedBy:
  - domain.scenario-management
  - domain.requirements
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - domain.autonomy
  - domain.collaboration-patterns
  - domain.disruption-review
  - domain.task-decomposition
  - domain.project-management
  - domain.governance-audit
implementedIn:
  - sdi-plugin/crates/daemon
dependsOn:
  - component.core
  - component.db
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.cli
  - component.mcp
  - component.web-dashboard
  - component.hooks-adapter
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: component.core
    type: depends-on
    note: 도메인 타입과 유효성 규칙을 sdi-core에서 빌려온다
  - to: component.db
    type: depends-on
    note: sdi-db를 통해 SQLite를 단독 소유한다
  - to: domain.scenario-management
    type: backed-by
  - to: domain.governance-audit
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

sdid(sdi-daemon 바이너리)는 장시간 실행되는 로컬 HTTP 서버다. SDI가 다루는 모든 상태 — 프로젝트, 플랜, 요구사항, 결정, 시나리오, 라운드, 태스크, 협업 패턴, 자율 정책, 실행 기록, 감사 로그, 리소스 클레임 — 를 SQLite에 단독으로 보관하고, 그 위에 읽기·쓰기 HTTP API를 올린다.

게이트 로직도 데몬이 집행한다. Plan 승인 게이트(시나리오 ≥ 1, GWT 완전, D8), Decision 4단계 협상 게이트(비평 없이 합의 불가, D20), CollaborationPattern 전환 게이트(D26 형상 검증, D27 pending→active), L5 잠금 해제 조건(reversal_plan 비어 있으면 L5 불허, D28), 리소스 클레임 충돌 감지(D29) 등 모든 SDI 불변식의 최종 집행자가 이 데몬이다.

태스크가 done/cancelled 상태로 전환될 때, 그 부모 라운드·플랜의 자식이 모두 종료 상태인지 확인해 자동으로 완료를 cascade한다.

SSE(Server-Sent Events) 이벤트 버스를 제공해 상태 변경을 실시간으로 웹 대시보드와 watch 명령에 흘린다.

## 경계와 의존

데몬은 로컬에서만 수신한다. Unix 도메인 소켓과 루프백 TCP(기본 포트는 설정 파일 또는 환경 변수로 지정, 포트 충돌 시 자동 증가)에만 바인딩하고, 외부로 나가는 네트워크 요청을 만들지 않는다. XDG 경로 불변식에 따라 소켓 파일과 pid 파일은 `~/.cache/sdi/`에, SQLite 데이터는 `~/.local/share/sdi/`에, 로그는 `~/.local/state/sdi/`에 둔다. 플러그인 설치 경로 아래에 이 파일들이 생기면 시작을 거부한다.

sdi-core(도메인 타입·불변식)와 sdi-db(저장소 어댑터)에 의존한다. CLI·MCP·웹·훅은 데몬에 의존하지만 데몬은 그것들에 역의존하지 않는다.

## 통신 패턴

axum 기반 HTTP REST API + SSE 스트림으로 통신한다. 클라이언트(CLI, MCP, 웹, 훅 어댑터)는 HTTP 요청을 통해 데이터를 읽고 쓰며, 실시간 변경 알림은 SSE 이벤트 버스를 구독해 받는다. Unix 도메인 소켓을 통한 통신도 지원해 TCP 포트 없이 로컬 프로세스 간 고속 통신이 가능하다.

tower-http의 ServeDir 미들웨어를 통해 웹 대시보드 SPA(plugin/web/dist) 정적 파일도 직접 서빙한다. 별도 웹 서버 없이 데몬 하나가 API와 SPA 모두를 제공하는 구조다.

## 하위 서브패키지 (책임 단위)

- **HTTP 라우터**: 플랜·요구사항·결정·시나리오·라운드·태스크·패턴·정책 CRUD 라우트와 게이트 로직.
- **SSE 이벤트 버스**: 상태 변경을 구독자에게 실시간 브로드캐스트.
- **게이트 집행**: D8 플랜 승인, D20 결정 협상, D26/D27 패턴 형상, D28 L5 잠금, D29 리소스 클레임 충돌.
- **완료 cascade**: 태스크 종료 시 부모 컨테이너 자동 완료 처리.
- **SPA 서빙**: ServeDir로 plugin/web/dist 정적 파일 제공.
- **감사 로그**: 모든 뮤테이션을 시간 순으로 append-only 기록.

## 미확정 (OPEN)

- [ ] OPEN: 기본 포트 번호와 포트 충돌 시 자동 증가 상한은 코드에서 확인 필요.
- [ ] OPEN: AgentRun↔Scenario 엣지는 현재 SDI_ACTIVE_SCENARIO 환경 변수로 임시 처리 중이며, 데몬이 이 엣지를 직접 관리하는 시점은 미정이다(CLAUDE.md 명시).
