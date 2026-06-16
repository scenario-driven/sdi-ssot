---
id: decision.dist-branch-release
kind: Decision
title: 빌드 산출물은 dist 브랜치 또는 GitHub Releases로 배포하고 사용자 머신에 Rust toolchain을 요구하지 않는다
purpose: "사용자가 SDI를 설치할 때 Rust 소스를 직접 빌드하게 할지, 사전 빌드된 바이너리를 배포할지"
definition: "SDI의 배포 단위는 빌드된 바이너리(sdi, sdid)와 플러그인 매니페스트·웹 번들이다. main 브랜치는 소스, dist 브랜치(또는 GitHub Releases tarball)는 빌드 산출물을 담는다. 사용자 머신에 Rust toolchain을 설치할 것을 요구하지 않는다."
relatesTo:
  - { to: domain.plugin-runtime, type: governs, note: "플러그인 설치·배포 방식을 이 결정이 정의한다" }
  - { to: platform.sdi, type: impacts, note: "사용자가 SDI를 설치하는 경로가 이 결정에 의해 규정된다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI는 Claude Code 플러그인이 본체다. 플러그인 마켓플레이스에서 설치되는 모델에서는, 사용자 머신에 언어 toolchain이 없어도 플러그인이 동작해야 한다. Rust 바이너리(`sdi`, `sdid`)를 소스로 배포하면 사용자가 `cargo build`를 직접 실행해야 하고, 이는 Rust toolchain 설치와 수 분의 빌드 시간을 요구한다.

한편 같은 repo에서 소스와 바이너리를 동시에 관리하면 repo 히스토리가 오염되는 문제가 있어, dist 브랜치 또는 GitHub Releases로 분리하는 패턴이 일반적이다(Biome, Deno, rust-analyzer 등의 선례가 있다).

## 결정 (Decision)

`main` 브랜치는 소스 코드만 담는다. 사전 빌드된 바이너리(`sdi`, `sdid`)와 웹 번들(`plugin/web/dist/`), 플러그인 매니페스트를 `dist` 브랜치 또는 GitHub Releases tarball로 배포한다. 두 방식 중 최종 선택은 구현 시점에 결정되며 현재 전략이 확정되지 않은 상태다.

플러그인 설치 게이트(`scripts/setup.cjs`, `plugin/adapters/shared/sdi-hooks.cjs`)는 설치 시점에 바이너리가 없으면 GitHub Releases에서 해당 플랫폼 tarball을 내려받아 `plugin/bin/`과 `plugin/daemon/bin/`에 배치한다(SDI_RELEASE_FETCH=1 경로). 로컬 개발 중에는 `target/release/sdi` 또는 `target/debug/sdi`가 폴백으로 사용된다.

## 근거와 결과 (Consequences)

빌드 산출물을 별도 브랜치/릴리즈로 분리하면 다음이 따라온다.

- 사용자 머신에 Rust toolchain 요구 없음: 플러그인 마켓플레이스를 통해 일반 개발자도 SDI를 설치할 수 있다.
- 소스와 바이너리의 대응이 원자적: 특정 GitHub Release tag가 어떤 소스 커밋에서 빌드됐는지 추적 가능하다.
- CI/CD 파이프라인이 명확: main 브랜치 푸시 → 빌드 → dist 브랜치 갱신 + GitHub Release 생성의 단방향 흐름.

대가는 배포 파이프라인을 별도로 구성해야 한다는 것이다. 그리고 dist 브랜치 전략의 세부 사항(어떤 파일을 어떻게 구성하는지)은 `docs/RELEASING.md`에 확정 기록될 예정이다.

<!-- provenance: sdi-plugin/CLAUDE.md "Distribution branch. Strategy not yet locked. Either a dist branch carrying built binaries + plugin manifest, or direct consumption from main with binaries fetched from GitHub Releases." sdi-plugin/docs/ARCHITECTURE.md "Why one repo" 섹션 3번. sdi-plugin/docs/ARCHITECTURE.md "Binary resolution" 섹션. -->
