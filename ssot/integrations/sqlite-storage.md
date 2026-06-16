---
id: integration.sqlite-storage
kind: Integration
title: SQLite 로컬 스토리지 연동
purpose: 모든 SDI 상태(플랜·시나리오·결정·라운드·태스크·CollaborationPattern·AutonomyPolicy)를 외부 서버 없이 사용자 머신의 로컬 SQLite 파일 하나에 영구 보존한다. FTS5 전문검색이 기본 내장되며, 벡터 임베딩 검색은 현 버전에서 제외되었다.
definition: "SDI 데몬(`sdid`)이 기동 시 `~/.local/share/sdi/sdi.db` 파일을 열고, `sdi-db` 크레이트가 `rusqlite + r2d2 커넥션 풀`로 SQLite에 접근한다. 14개 순서형 마이그레이션으로 스키마를 관리한다. FTS5 가상 테이블이 시나리오·지식 전문검색을 제공하고, MCP `search_knowledge` 도구가 이 인덱스를 통해 RAG를 구현한다."
integratesWith: []
implementedIn:
  - sdi-plugin/crates/db/src/lib.rs
  - sdi-plugin/crates/db/src/schema.rs
  - sdi-plugin/crates/db/src/pool.rs
  - sdi-plugin/crates/db/src/paths.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/002_disruption_reviews.sql
  - sdi-plugin/crates/db/src/migrations/003_collab.sql
  - sdi-plugin/crates/db/src/migrations/004_runs_hierarchy.sql
  - sdi-plugin/crates/db/src/migrations/005_usage.sql
  - sdi-plugin/crates/db/src/migrations/006_v04_multi_agent.sql
  - sdi-plugin/crates/db/src/migrations/007_v05_pattern_enforcement.sql
  - sdi-plugin/crates/db/src/migrations/008_project_metadata.sql
  - sdi-plugin/crates/db/src/migrations/009_direct_provenance_backfill.sql
  - sdi-plugin/crates/db/src/migrations/010_short_code_scope.sql
  - sdi-plugin/crates/db/src/migrations/011_direct_provenance_backfill_rerun.sql
  - sdi-plugin/crates/db/src/migrations/012_scenario_retire.sql
  - sdi-plugin/crates/db/src/migrations/013_decision_supersede_when.sql
  - sdi-plugin/crates/db/src/migrations/014_round_baseline.sql
impacts:
  - domain.scenario-management
  - domain.requirements
  - domain.task-decomposition
  - domain.planning
  - domain.round-execution
  - domain.decision-negotiation
  - domain.collaboration-patterns
  - domain.autonomy
  - domain.knowledge-rag
  - domain.project-management
relatesTo:
  - to: domain.scenario-management
    type: realizes
    note: 시나리오 엔티티의 영구 저장소다
  - to: domain.knowledge-rag
    type: realizes
    note: FTS5 인덱스가 knowledge RAG 검색의 물리적 구현이다
  - to: domain.planning
    type: realizes
    note: 플랜·요구사항이 이 SQLite에 저장된다
  - to: domain.decision-negotiation
    type: realizes
    note: Decision append-only 행이 이 스토리지에 축적된다
  - to: domain.collaboration-patterns
    type: realizes
    note: CollaborationPattern 행과 produced_via_pattern_id FK가 이 스토리지에 존재한다
  - to: domain.autonomy
    type: realizes
    note: AutonomyPolicy 행이 이 스토리지에 저장된다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 SQLite와 그 Rust 바인딩인 rusqlite, 그리고 커넥션 풀링을 위한 r2d2다. 이 세 외부 의존이 SDI의 유일한 퍼시스턴스 레이어다. 외부 서버·클라우드 동기화·외부 네트워크 연결은 없다(PRD §8 명시적 비목표).

데이터 파일 위치는 `~/.local/share/sdi/sdi.db`다. 데몬(`sdid`)이 시작 시 이 경로를 열어 커넥션 풀을 생성하고, 스키마 마이그레이션을 순서대로 적용한 뒤 HTTP 서빙을 시작한다. CLI·MCP 서버는 직접 SQLite에 접근하지 않고 반드시 데몬 HTTP API를 통해서만 데이터를 읽고 쓴다.

FTS5 가상 테이블이 시나리오·지식 엔트리의 전문검색 인덱스를 제공한다. MCP의 `search_knowledge`·`search_scenarios` 도구가 이 인덱스를 통해 LLM에게 RAG 결과를 돌려준다. 벡터 임베딩 기반 검색은 현 버전에서 보류 상태다(`sdi-db` 크레이트 설명 "FTS5 keyword search; vector search deferred" 명시).

스키마는 14개의 번호 순서형 마이그레이션 SQL 파일로 관리된다. 마이그레이션은 데몬 기동 시 자동으로 적용되며 건너뜀을 허용하지 않는다. 각 마이그레이션의 이름이 그 시점의 도메인 결정을 반영한다: `001_core`(핵심 엔티티+FTS5), `003_collab`(CollaborationPattern D22), `006_v04_multi_agent`(multi-agent 지원), `007_v05_pattern_enforcement`(D26 패턴 무결성 게이트), `009_direct_provenance_backfill`·`011`(D23 direct 패턴 소급 적용) 등.

## 구현 위치 (provenance)

스토리지 어댑터 크레이트의 진입점은 `crates/db/src/lib.rs`다. 경로 해석은 `paths.rs`(XDG 경로 + 환경 변수 오버라이드 지원), 커넥션 풀은 `pool.rs`, 스키마 마이그레이션은 `schema.rs`가 담당한다. 실제 CRUD 구현은 `crates/db/src/repo/` 하위 모듈들에 있으며, 상위 크레이트(`cli`·`daemon`·`mcp`)는 이 크레이트를 통해서만 스토리지에 접근한다.

SQLite는 `bundled` 피처로 링크되어 시스템 SQLite 설치에 의존하지 않는다. `chrono`·`serde_json` 피처도 활성화되어 Rust 타입과 SQLite 컬럼 간 직렬화를 처리한다.

## 불변식

- 데이터 파일은 `~/.local/share/sdi/`에만 위치해야 한다. `~/.claude/plugins/` 하위 경로에 데이터 파일이 생기면 데몬이 시작을 거부한다(LM-8 경로 분리 불변식). `sdi doctor`도 이 위반을 종료 코드 1로 감지한다.
- CLI·MCP는 SQLite에 직접 접근하지 않는다. 모든 데이터 접근은 데몬 HTTP API를 통해서만 이뤄진다.
- 마이그레이션은 항상 순서대로 적용된다. 건너뜀이나 역방향 적용을 허용하지 않는다.
- 플러그인 설치/제거 사이클은 `~/.local/share/sdi/` 데이터를 건드리지 않는다. 데이터는 바이너리 교체와 무관하게 보존된다.
- FTS5 인덱스는 항상 본체 테이블과 동기화 상태여야 한다. 인덱스 미러 갱신은 데몬 쓰기 경로가 담당한다.

## 영향 범위

SQLite 파일이 손상되거나 마이그레이션이 실패하면 데몬이 기동하지 않는다. 데몬이 없으면 MCP·CLI·훅의 모든 데이터 의존 기능이 멈춘다. 즉 이 연동은 SDI의 전체 상태 레이어이며, 다른 모든 연동이 이 위에서 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: 벡터 임베딩 검색 도입 시 sqlite-vec 등 외부 확장의 bundled 여부와 XDG 경로 정책 재확인 필요.
- [ ] OPEN: WAL 모드 활성화 여부와 multi-session 동시 접근(D29) 시나리오에서의 SQLite 락 정책 명시 필요.
