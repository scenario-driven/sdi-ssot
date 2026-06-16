---
id: decision.plugin-is-the-product
kind: Decision
title: Claude Code 플러그인이 SDI의 본체이고 Rust workspace는 같은 repo 안의 두 번째 뷰다
purpose: "SDI를 어디서 시작하는 제품으로 규정할지 — 독립 CLI/서버 도구로 갈지, Claude Code 플러그인을 본체로 할지"
definition: "sdi-plugin은 Claude Code 플러그인이 본체다. plugin 셸(plugin/)과 Rust workspace(crates/)는 같은 소스 트리의 두 가지 뷰이며, 별도 산출물이 아니다. daemon·MCP·대시보드 SPA는 모두 그 플러그인 안의 구성요소다."
relatesTo:
  - { to: domain.plugin-runtime, type: governs, note: "이 결정이 플러그인 런타임 도메인의 존재 근거다" }
  - { to: platform.sdi, type: impacts, note: "SDI 제품 전체의 배포·설치·런타임 모델을 이 결정이 규정한다" }
  - { to: domain.project-management, type: impacts, note: "프로젝트가 플러그인 설치 경로에 종속된다" }
supersedes: []
implementedIn:
  - sdi-plugin/plugin
  - sdi-plugin/crates
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI를 설계할 때 배포·런타임 모델의 첫 갈림길은 "독립 CLI 도구로 만들어 사용자가 직접 설치하게 할 것인가, Claude Code 플러그인으로 만들어 플러그인 마켓플레이스를 통해 배포할 것인가"였다.

Claude Code 플러그인 스펙은 마켓플레이스 플러그인을 `~/.claude/plugins/cache/<plugin>/`으로 복사한다. 플러그인 디렉토리 바깥에 대한 경로 트래버설(`../crates`)은 설치 후 동작하지 않는다. 따라서 런타임에 필요한 모든 것(실행 바이너리, 웹 번들, hook 스크립트)이 플러그인 디렉토리 안에 있어야 한다.

이 제약은 동시에 하나의 기회이기도 하다. Rust workspace를 플러그인 repo 안에 함께 두면, 소스와 바이너리의 대응 관계가 원자적으로 유지되고 PR 리뷰에서도 변경사항이 한 repo에 모인다.

## 결정 (Decision)

sdi-plugin repo를 Claude Code 플러그인이 본체인 단일 repo로 정한다. `plugin/` 셸(plugin 매니페스트, hook 스크립트, 슬래시 명령, 스킬, MCP 설정, 설치 게이트)과 `crates/` Rust workspace(cli/daemon/mcp/core/db 크레이트)는 같은 소스 트리 안에 공존하며, 독립적으로 배포되는 별도 산출물이 아니다.

daemon(`sdid`)이 SQLite 상태를 보유하고 HTTP + unix socket을 서빙한다. MCP 서버는 `sdi mcp` 서브커맨드로 실행된다. 대시보드 SPA는 `plugin/web/`에서 Vite/React로 빌드되어 sdid가 tower-http ServeDir로 직접 서빙한다. 이 구성요소 전체가 플러그인의 일부다.

## 근거와 결과 (Consequences)

플러그인이 본체라는 결정에서 여러 파생 결정이 따라온다.

- 단일 Rust workspace(decision.single-rust-workspace): 크레이트들이 같은 repo 안에 있어야 하므로 workspace resolver를 쓰는 단일 Cargo.toml이 합리적이다.
- XDG 경로 분리(decision.xdg-data-paths): 플러그인 디렉토리가 재설치 시 덮어씌워지므로 사용자 데이터는 반드시 XDG 경로에 보관해야 한다.
- dist 브랜치 배포(decision.dist-branch-release): plugin 디렉토리 구조와 바이너리를 같이 배포하려면 소스 브랜치와 분리된 배포 브랜치가 필요하다.
- daemon이 SPA 서빙(decision.daemon-serves-spa): SPA가 플러그인 디렉토리 안에 있으므로 daemon이 그 경로를 알고 ServeDir로 서빙하는 것이 자연스럽다.

대가는 Rust toolchain과 Node.js 환경이 플러그인 개발 시에 모두 필요하다는 것이다. 그러나 사용자 머신에는 빌드된 바이너리와 웹 번들만 배포되므로 Rust toolchain 설치를 요구하지 않는다.

<!-- provenance: sdi-plugin/CLAUDE.md "Identity" 섹션 "The plugin is not a thin wrapper around a separate Rust project — they are the same artifact, two views of the same source tree". sdi-plugin/docs/ARCHITECTURE.md "Why one repo" 섹션. sdi-plugin/docs/PRD.md §5.1 -->
