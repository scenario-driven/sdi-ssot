---
id: invariant.db-single-source-of-truth
kind: Invariant
title: SQLite DB가 모든 SDI 상태의 단일 진실 공급원이다
definition: "SDI의 모든 상태(Plan·Scenario·Task·Decision·Round·AutonomyPolicy·CollaborationPattern·AgentNote·AgentSpec)는 단일 SQLite 파일에만 저장되며, 이 파일이 유일한 진실 공급원이다. 메모리 캐시, 파일 시스템 사이드카, 원격 DB는 상태의 권위 있는 소스가 아니며, 항상 SQLite 내용과 일치해야 한다."
governs:
  - platform.sdi
  - domain.project-management
  - domain.scenario-management
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - concept.plan
  - concept.scenario
  - concept.task
  - concept.decision
implementedIn:
  - "sdi-plugin/crates/db/src/lib.rs"
  - "sdi-plugin/crates/daemon/src/main.rs"
  - "sdi-plugin/docs/ARCHITECTURE.md"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 데몬(`sdid`)은 모든 상태를 `~/.local/share/sdi/sdi.db` 단일 SQLite 파일에 저장한다. 어떤 서비스나 에이전트도 이 파일을 직접 쓰지 않고 반드시 데몬의 HTTP API를 통해 읽고 쓴다. 플러그인 훅은 데몬에 HTTP/소켓 요청을 보내는 방식으로만 상태에 접근한다. CLI(`sdi`)와 MCP 서버도 모두 데몬을 경유한다.

Clawket의 `db-is-sot-plan-markdown-view` 불변식을 계승한다. 이 불변식은 "Markdown 파일 같은 파생 표현은 뷰(view)이지 진실이 아니다"를 명시한다. 대시보드 SPA, `sdi doctor` 출력, MCP 응답은 모두 SQLite 내용을 읽어 표현한 것이며 독립적인 진실이 아니다.

SQLite의 ACID 보장이 단일 데몬 인스턴스(`invariant.daemon-single-instance`)와 결합하여 여러 Claude Code 세션이 같은 계획을 작업해도 데이터 일관성이 유지된다(D29 Multi-session resource claims의 전제).

## 깨지면 무슨 일이 일어나나

상태가 두 곳 이상에 저장되면 어느 것이 현재 진실인지 알 수 없다. 예를 들어 Plan의 현재 lifecycle을 파일 시스템 마크다운에서 읽으면, 데몬이 approve 요청을 처리하는 동안 파일이 아직 갱신되지 않은 stale 상태를 읽을 수 있다. 복수 에이전트가 서로 다른 소스에서 상태를 읽으면 어떤 에이전트는 Round를 active로 보고 다른 에이전트는 planning으로 보는 불일치가 생긴다. 이 불일치 상태에서 Task를 분해하거나 회귀 검증을 시작하면 결과가 신뢰할 수 없다.

## 코드에서 어떻게 강제되나

데몬의 모든 핸들러(`crates/daemon/src/router/`)는 rusqlite 연결을 통해서만 데이터를 읽고 쓴다. 데몬 외부에서 DB 파일에 직접 접근하는 코드가 없으며, 플러그인 훅 코드(`plugin/adapters/shared/sdi-hooks.cjs`)는 데몬의 HTTP/소켓 API만 사용한다. `crates/db/src/lib.rs`의 connection pool이 단일 접속 지점을 관리하며, SQLite의 WAL 모드가 단일 writer + 복수 reader를 안전하게 처리한다. DB 경로는 `crates/db/src/paths.rs`에서 XDG 규칙에 따라 단일하게 해석된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(SQLite 단일 진실 공급원 정책의 결정 엔트리) 연결 필요
