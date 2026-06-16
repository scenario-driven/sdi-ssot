---
id: integration.github-releases
kind: Integration
title: GitHub Releases 바이너리 배포
purpose: 사용자 머신에 Rust 툴체인 없이도 SDI를 사용할 수 있도록 — 컴파일된 `sdi`·`sdid` 바이너리와 빌드된 대시보드 SPA를 GitHub Releases + `dist` 브랜치에 올려두고, Claude Code 마켓플레이스가 그 브랜치를 플러그인 설치 소스로 삼는다.
definition: "SDI Rust 워크스페이스 단일 버전 태그(`v*.*.*`)를 기준으로 GitHub Actions가 네 개 타깃(macOS + Linux × x86_64 + aarch64)의 바이너리를 빌드해 GitHub Release에 첨부하고, `dist` 브랜치에 `plugin/bin/`·`plugin/daemon/bin/`을 포함한 배포 트리를 force-push한다. 플러그인 설치 게이트는 이 결과물이 이미 배치된 경로에서 바이너리를 탐색한다."
integratesWith: []
implementedIn:
  - sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs
  - sdi-plugin/plugin/scripts/setup.cjs
  - sdi-plugin/docs/RELEASING.md
impacts:
  - domain.plugin-runtime
  - domain.project-management
relatesTo:
  - to: domain.plugin-runtime
    type: realizes
    note: 배포된 바이너리가 플러그인 런타임(sdid 데몬·sdi CLI)의 물리적 공급원이다
  - to: domain.project-management
    type: relates-to
    note: 버전 태그와 릴리스 체크리스트가 프로젝트 관리 도메인과 연결된다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 GitHub의 릴리스 배포 인프라다. SDI는 단일 Rust 워크스페이스(`crates/cli`·`daemon`·`mcp`·`core`·`db`) + 플러그인 셸(`plugin/`) + 대시보드 SPA(`plugin/web/`)를 하나의 버전으로 묶어 릴리스한다.

배포 흐름은 두 결과물로 나뉜다.

첫째, **GitHub Release**: `v*.*.*` 태그 push 시 GitHub Actions가 네 개 플랫폼 바이너리(`sdi`·`sdid`)를 빌드해 GitHub Release 자산으로 첨부한다. 각 바이너리는 체크섬과 함께 배포될 수 있다.

둘째, **`dist` 브랜치**: 같은 CI 파이프라인이 빌드된 바이너리를 `plugin/bin/`(`sdi`)과 `plugin/daemon/bin/`(`sdid`)에 두고 `dist` 브랜치를 force-push한다. Claude Code 마켓플레이스는 이 브랜치를 플러그인 트리로 복사해 `~/.claude/plugins/cache/`에 설치한다. `main` 브랜치는 소스 전용이므로 바이너리는 `.gitignore`로 추적하지 않는다.

`sdi-desktop` 레포는 별도 레포이며 독립 릴리스 주기를 갖는다. 대시보드 SPA(`plugin/web/`)는 이 워크스페이스 안에 있으므로 같은 플러그인 태그와 함께 배포된다.

## 구현 위치 (provenance)

설치 게이트의 바이너리 탐색 로직은 `plugin/adapters/shared/sdi-hooks.cjs`의 `resolveSdiBin`·`resolveSdidBin` 함수에 있다. 탐색 우선순위는 다음 순서다: `SDI_BIN` 환경 변수 → `<pluginRoot>/bin/sdi`(릴리스 번들 레이아웃) → 워크스페이스 `target/release/sdi` → `target/debug/sdi` → `PATH`의 `sdi`. 릴리스 번들이 설치된 환경에서는 두 번째 경로가 히트한다.

대시보드 SPA 번들은 `resolveSdiWebDist`가 탐색한다: `SDI_WEB_DIST` 환경 변수 → `<pluginRoot>/web/dist` → 워크스페이스의 `plugin/web/dist`. `dist`가 없고 소스만 있으면 `buildable` 상태를 반환하고, 없으면 `absent`다.

수동 설치나 CI 진입점은 `plugin/scripts/setup.cjs`이며 같은 `ensureInstalled` 코드 경로를 공유한다.

릴리스 순서와 체크리스트는 `docs/RELEASING.md`가 정의한다: 워크스페이스 바이너리 빌드·GitHub Release 생성 → `dist` 브랜치 갱신 → `sdi-desktop`은 별도.

## 불변식

- `main` 브랜치는 소스 전용이다. `plugin/bin/`과 `plugin/daemon/bin/`은 `.gitignore`로 추적하지 않는다. 바이너리는 `dist` 브랜치에만 존재한다.
- 워크스페이스 전체가 단일 버전을 공유한다(`Cargo.toml [workspace.package].version`). 별도 컴포넌트 버전이 없다.
- 사용자 데이터(`~/.local/share/sdi/`)는 플러그인 설치/제거 주기와 무관하게 보존되어야 한다(LM-8 경로 분리 불변식). `dist` 브랜치 갱신이 XDG 사용자 데이터를 건드리는 일은 없다.
- 릴리스 후 퍼블릭 태그는 삭제하지 않는다. 와이어 형상 호환성 문제가 생기면 수정 패치 릴리스를 내고, 이전 릴리스를 GitHub에서 "pre-release"로 마킹한다.

## 영향 범위

이 연동이 실패하면 신규 설치 또는 업데이트 시 바이너리가 없어 `sdi doctor`가 실패하고 SessionStart 훅이 플러그인을 동작시키지 못한다. 이미 설치된 사용자는 기존 바이너리로 동작을 계속할 수 있다. `dist` 브랜치가 손상되면 마켓플레이스 설치 흐름 전체가 끊긴다. `sdi-desktop`은 독립 릴리스 주기이므로 이 연동의 직접 영향권이 아니다.

## 미확정 (OPEN)
- [ ] OPEN: GitHub Actions 릴리스 워크플로우 파일(`.github/workflows/release.yml`)이 현재 존재하는지, 또는 미래 구현 예정인지 확인 필요.
- [ ] OPEN: 체크섬 파일(SHA256SUMS) 제공 여부와 설치 게이트의 무결성 검증 정책이 Clawket와 달리 아직 구체화되지 않음.
