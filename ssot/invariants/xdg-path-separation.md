---
id: invariant.xdg-path-separation
kind: Invariant
title: 데이터·캐시·설정·로그 경로를 XDG 규칙으로 분리한다
definition: "DB는 데이터 디렉터리(~/.local/share/sdi/), 런타임 상태는 캐시 디렉터리(~/.cache/sdi/), 설정은 설정 디렉터리(~/.config/sdi/), 로그는 상태 디렉터리(~/.local/state/sdi/)로 XDG 규칙에 따라 분리되며, 각 경로는 SDI_HOME 환경변수로 재정의할 수 있고 플러그인 재설치가 사용자 데이터를 건드리지 않는다."
governs:
  - domain.plugin-runtime
  - domain.project-management
  - concept.project
implementedIn:
  - "sdi-plugin/crates/db/src/paths.rs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
  - "sdi-plugin/docs/ARCHITECTURE.md"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI가 다루는 네 용도 영역(데이터 / 캐시·런타임 / 설정 / 로그)은 XDG 표준 디렉터리로 엄격히 분리된다. 구체적으로 SQLite DB는 `~/.local/share/sdi/`에, 소켓·PID·포트 파일 등 런타임 캐시는 `~/.cache/sdi/`에, 사용자 설정은 `~/.config/sdi/`에, 훅 감사 로그는 `~/.local/state/sdi/hook.log`에 위치한다. 테스트 격리가 필요한 경우에는 `SDI_HOME` 환경변수로 루트를 재정의한다.

이 분리의 핵심 효과는 **플러그인 재설치 안전성**이다. Claude Code 플러그인 캐시 디렉터리(`~/.claude/plugins/cache/sdi/`)는 `/plugin install` 명령이 언제든 덮어쓸 수 있고, 해당 트리 아래에 사용자 데이터가 있으면 재설치 시 조용히 소실된다. XDG 경로 분리는 이 소실 경로를 구조적으로 차단한다.

네 경로 간 역할이 뒤섞여서는 안 된다. 특히 플러그인 코드가 XDG 경로에 직접 쓸 수 있는 유일한 채널은 `appendHookLog()`를 통한 상태 경로 단방향 기록뿐이며, 데이터·캐시·설정 경로에 대한 모든 쓰기는 데몬을 통해서만 이루어진다.

## 깨지면 무슨 일이 일어나나

가장 파괴적인 위반은 SQLite DB 또는 설정 파일이 플러그인 디렉터리 하위에 위치하는 경우다. 이 상태에서 플러그인이 업데이트되거나 재설치되면 사용자의 모든 Plan·Scenario·Decision·Round 이력이 경고 없이 삭제된다. 소실된 이력은 시나리오 기반 회귀 검증의 기반 자체가 없어지는 것이므로 SDI의 핵심 가치 명제가 통째로 사라진다. 캐시 경로 오염의 경우 캐시 정리 스크립트가 소켓 파일이나 PID 파일을 삭제하면서 데몬이 예상치 않게 종료된다. 로그 경로 오염은 감사 추적이 캐시 만료와 함께 소실되어 `sdi bypass arm` 남용을 추적할 수 없게 만든다.

## 코드에서 어떻게 강제되나

데몬(`sdid`)이 시작 시점에 `crates/db/src/paths.rs`의 `ensure_no_plugin_overlap` 함수를 호출하여 데이터·캐시·설정·상태·DB 파일 경로 각각이 `~/.claude/plugins/` 하위로 해석되지 않는지 검사한다. 단 하나의 경로라도 겹치면 데몬이 시작을 거부한다. `sdi doctor` 하위 명령도 동일 검사를 수행하여 exit code 1로 사용자에게 위반을 알린다. 플러그인 훅 코드(`plugin/adapters/shared/sdi-hooks.cjs`)는 XDG 경로에 직접 파일을 쓰지 않으며 오직 `appendHookLog()` 단일 채널만 허용한다.

`CLAWKET_ALLOW_PLUGIN_OVERLAP=1` 환경변수가 유일한 우회 수단이며, 이는 의도적으로 비표준 이름을 사용하여 의도치 않은 설정을 방지한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(Clawket LM-8 invariant 계승 결정의 SDI 측 결정 엔트리) 연결 필요
