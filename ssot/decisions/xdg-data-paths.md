---
id: decision.xdg-data-paths
kind: Decision
title: 사용자 데이터는 XDG 경로에 두고 플러그인 디렉토리 아래에는 절대 놓지 않는다
purpose: "SDI의 사용자 데이터(SQLite, 소켓, 설정, 로그)를 어디에 저장할지 — 플러그인 디렉토리 안에 둘지, 표준 XDG 경로에 분리해 둘지"
definition: "SQLite 데이터는 ~/.local/share/sdi/, 캐시(소켓·pid·포트)는 ~/.cache/sdi/, 설정은 ~/.config/sdi/, 상태(로그)는 ~/.local/state/sdi/ 에 둔다. 플러그인 디렉토리(~/.claude/plugins/...)에는 바이너리·매니페스트·웹 번들만 존재하고 사용자 데이터는 절대 들어가지 않는다."
relatesTo:
  - { to: domain.plugin-runtime, type: governs, note: "플러그인 런타임이 데이터를 어디에 쓸 수 있는지를 이 결정이 제한한다" }
  - { to: platform.sdi, type: impacts, note: "설치·재설치·삭제 시 사용자 데이터가 보전되는 근거가 이 결정이다" }
  - { to: domain.governance-audit, type: impacts, note: "감사 로그의 기록 위치도 XDG state 경로로 고정된다" }
supersedes: []
implementedIn:
  - sdi-plugin/crates/daemon
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Claude Code 플러그인은 마켓플레이스에서 `~/.claude/plugins/cache/<plugin>/`으로 복사된다. `/plugin install` 명령은 이 디렉토리를 재생성할 수 있다. 만약 SQLite 데이터베이스나 설정 파일이 플러그인 디렉토리 안에 있다면, 플러그인 업데이트나 재설치 시 사용자 데이터가 조용히 삭제될 위험이 있다. 이는 Clawket(선행 도구)이 설계 당시부터 인식하고 해결했던 불변식이었으며(LM-8이라 명명), SDI는 이를 계승한다.

## 결정 (Decision)

사용자 데이터를 XDG Base Directory Specification 경로로 고정한다.

| 영역 | 경로 |
|---|---|
| 데이터(SQLite) | `~/.local/share/sdi/` |
| 캐시(소켓·pid·포트 파일) | `~/.cache/sdi/` |
| 설정 | `~/.config/sdi/` |
| 상태(로그) | `~/.local/state/sdi/` |

daemon(`sdid`)은 시작 시 이 다섯 경로 중 하나라도 `~/.claude/plugins/` 아래로 해석되면 즉시 시작을 거부한다(`paths::ensure_no_plugin_overlap`). `sdi doctor`는 이 검사를 재실행하여 위반 시 exit code 1을 반환한다. 플러그인 설치 게이트는 `sdi`, `sdid` 바이너리를 `~/.claude/plugins/sdi-*/bin/`에만 쓸 수 있다.

## 근거와 결과 (Consequences)

XDG 경로로 분리함으로써 세 가지가 보장된다.

첫째, 플러그인 재설치·업데이트 안전성: 플러그인 디렉토리가 덮어씌워져도 사용자의 시나리오·플랜·결정·감사 로그는 별도 경로에 살아있다.

둘째, 표준 경로 준수: OS 패키지 관리자, 백업 도구, dotfile 관리가 XDG 경로를 기준으로 동작한다. 사용자가 `~/.local/share/sdi/`를 백업하면 모든 SDI 데이터를 이전·복원할 수 있다.

셋째, 구조적 제어: daemon이 시작 시 이 불변식을 강제 검사하므로, 설정 오류나 경로 변수 오버라이드로 인한 데이터 혼재를 조용히 허용하지 않는다.

이 결정은 Clawket LM-8 invariant의 직접 계승이다. SDI가 Clawket을 대체하더라도 이 경로 설계 원칙은 그대로 유지된다.

<!-- provenance: sdi-plugin/CLAUDE.md "XDG path invariant (carried from Clawket LM-8)" 섹션. sdi-plugin/docs/ARCHITECTURE.md "Data location (LM-8 invariant)" 섹션. scenario-driven/CLAUDE.md "사용자 데이터 경로 (XDG)" 테이블. crates/daemon의 paths::ensure_no_plugin_overlap 구현 예정. -->
