---
id: decision.single-rust-workspace
kind: Decision
title: CLI·daemon·MCP·core·db를 단일 Rust workspace 하나에 둔다
purpose: "sdi 관련 Rust 크레이트를 여러 repo에 분산할지, 하나의 workspace로 묶을지"
definition: "crates/ 아래 cli/daemon/mcp/core/db 다섯 크레이트를 단일 Cargo workspace(resolver=2)로 관리한다. 새 크레이트는 crates/ 아래에 추가하고 workspace.members에 등록한다."
relatesTo:
  - { to: domain.plugin-runtime, type: impacts, note: "workspace 구조가 플러그인 런타임의 바이너리 구성 방식을 결정한다" }
  - { to: platform.sdi, type: impacts, note: "sdi/sdid 두 바이너리가 이 workspace에서 빌드된다" }
supersedes: []
implementedIn:
  - sdi-plugin/Cargo.toml
  - sdi-plugin/crates
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI의 Rust 구성 요소는 기능 레이어별로 다섯 크레이트로 분리된다. cli(사용자 바이너리 `sdi`), daemon(상주 서비스 `sdid`), mcp(MCP 서버 라이브러리 — cli에 임베드), core(도메인 모델 + 리포지토리 트레이트), db(rusqlite + sqlite-vec 어댑터). 이 다섯을 어디에 두느냐가 선택지였다.

가장 단순한 대안은 각 크레이트를 별도 repo로 분리하는 것이다. Clawket은 CLI·daemon·web 등이 7개 repo에 분산되어 있었고, 그로 인해 cross-repo 의존성 업데이트·버전 맞춤·PR 연동이 복잡했다. SDI는 이 복잡성을 제거하는 것을 명시적 목표로 삼았다.

## 결정 (Decision)

다섯 크레이트를 `crates/` 아래에 모두 두고 workspace 루트의 `Cargo.toml` 하나로 관리한다. workspace resolver는 2를 사용한다. 각 크레이트 `Cargo.toml`은 `version.workspace = true` 등으로 workspace 공통 메타데이터를 참조한다. 새 크레이트는 항상 `crates/` 아래에 추가하고 같은 변경에서 `[workspace].members`를 갱신한다.

바이너리는 두 개다. `cli` 크레이트에서 `sdi`, `daemon` 크레이트에서 `sdid`가 나온다. 라이브러리 크레이트는 `src/lib.rs`를 노출한다.

## 근거와 결과 (Consequences)

단일 workspace로 묶으면 다음이 따라온다.

- 크레이트 간 의존성을 path 의존(`core = { path = "../core" }`)으로 표현할 수 있어 별도 배포 없이 로컬에서 즉시 빌드가 된다.
- `cargo build --workspace` 한 번으로 두 바이너리를 모두 얻는다. CI 파이프라인이 단순해진다.
- 버전 번호·메타데이터를 `workspace.package`에서 한 번만 정의하므로 버전 불일치가 없다.
- 크레이트 하나의 변경이 바이너리 전체에 영향을 주는지 여부를 같은 PR에서 `cargo check --workspace`로 즉시 확인할 수 있다.

대가는 repo 크기가 커지고 `cargo build`에 포함되는 크레이트가 늘어난다는 것이다. SDI의 규모(5 크레이트)에서는 이 비용이 다중 repo 유지 비용보다 작다고 판단했다.

<!-- provenance: sdi-plugin/CLAUDE.md "Repo conventions — Single Rust workspace, resolver = 2. New crates go under crates/". sdi-plugin/docs/ARCHITECTURE.md Layout 섹션. sdi-plugin/Cargo.toml -->
