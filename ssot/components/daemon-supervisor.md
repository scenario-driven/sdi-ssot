---
id: component.daemon-supervisor
kind: SystemComponent
title: SDI 데몬 수퍼바이저 (플러그인 내 sdid 라이프사이클 관리)
purpose: 플러그인 설치 경로 아래에 sdid 바이너리를 배치하고, 플러그인 활성화 시 데몬 프로세스를 기동·감시·재시작하는 역할을 담당한다. 사용자가 별도로 데몬을 수동 관리하지 않아도 플러그인 사용 중 데몬이 항상 떠 있도록 보장한다.
realizedBy:
  - domain.plugin-runtime
implementedIn:
  - sdi-plugin/plugin/daemon
  - sdi-plugin/plugin/bin
dependsOn:
  - component.daemon
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - component.hooks-adapter
  - component.web-dashboard
  - component.mcp
relatesTo:
  - to: component.plugin-shell
    type: belongs-to
    note: plugin/daemon/ 과 plugin/bin/ 이 플러그인 셸 안에 있다
  - to: component.daemon
    type: depends-on
    note: sdid 바이너리를 기동·감시·재시작하는 것이 이 컴포넌트의 본무다
  - to: domain.plugin-runtime
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

데몬 수퍼바이저는 플러그인 셸 안에서 sdid 바이너리의 생명주기를 관리하는 레이어다. 구체적으로 `plugin/bin/` 아래에 빌드된 `sdi`와 `sdid` 바이너리가 위치하며, `plugin/daemon/bin/`이 데몬 관련 런타임 파일의 보관 위치가 된다.

플러그인이 Claude Code에 설치되면 sdid 프로세스가 자동으로 기동되어야 한다. 이미 실행 중인 데몬이 있으면 중복 기동을 피하고, 데몬이 예상치 않게 종료되면 재기동한다. 이 감시·재시작 로직이 데몬 수퍼바이저의 핵심 역할이다.

현재 `plugin/bin/`과 `plugin/daemon/bin/`에는 `.gitkeep` 파일만 있으며, 실제 바이너리는 빌드 또는 설치 시점에 이 경로에 배치된다. 즉 이 컴포넌트는 "바이너리가 어디에 놓이는가"와 "어떻게 기동되는가"를 정의하는 구조적 슬롯이다.

## 경계와 의존

데몬 수퍼바이저는 sdid(sdi-daemon 크레이트)에 의존한다. sdid가 실제 상태 관리와 HTTP API를 담당하고, 이 컴포넌트는 그 프로세스를 띄우고 유지하는 역할만 한다. 역방향으로 sdid는 수퍼바이저에 의존하지 않는다 — sdid는 어떤 방식으로든(직접 실행, 수퍼바이저, CLI `sdi daemon start`) 기동될 수 있다.

XDG 경로 불변식에 따라 데몬 pid 파일·소켓·포트 파일은 `~/.cache/sdi/`에 위치한다. 플러그인 셸 경로(`~/.claude/plugins/sdi-*/`) 아래에 있는 것은 바이너리 파일뿐이며, 런타임 상태 파일은 XDG 경로에 간다.

## 통신 패턴

수퍼바이저는 os 프로세스 관리(fork/exec/pid 감시)로 sdid와 통신한다. 데몬이 정상 동작 중인지 확인하려면 데몬의 헬스 엔드포인트를 HTTP로 폴링한다.

## 하위 서브패키지 (책임 단위)

- **plugin/bin/**: sdi, sdid 바이너리 배치 위치(빌드/배포 시 채워짐).
- **plugin/daemon/bin/**: 데몬 관련 런타임 지원 파일 위치.
- **기동·감시 로직**: 플러그인 활성화 트리거에 따른 sdid 프로세스 시작, 헬스 체크, 재시작(구현 형식 미확정).

## 미확정 (OPEN)

- [ ] OPEN: 플러그인 활성화 트리거가 SessionStart 훅인지 별도 설치 스크립트인지 확인 필요.
- [ ] OPEN: 데몬 감시·재시작 로직의 구현 형식(Shell 스크립트, Node.js 스크립트, 또는 sdi CLI 명령)이 미정이다. 현재 plugin/daemon/bin/은 비어 있다.
