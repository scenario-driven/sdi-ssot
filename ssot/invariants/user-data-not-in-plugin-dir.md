---
id: invariant.user-data-not-in-plugin-dir
kind: Invariant
title: 사용자 데이터는 플러그인 디렉터리 안에 있어서는 안 된다
definition: "SDI가 관리하는 모든 사용자 데이터(SQLite DB, 소켓, 로그, 설정)는 ~/.claude/plugins/ 하위 경로에 위치해서는 안 된다. 플러그인이 쓸 수 있는 유일한 위치는 플러그인 루트 내 버전 마커와 XDG 상태 경로의 훅 감사 로그뿐이다."
governs:
  - domain.plugin-runtime
  - domain.project-management
  - concept.project
implementedIn:
  - "sdi-plugin/crates/db/src/paths.rs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
  - "sdi-plugin/CLAUDE.md"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

Claude Code 플러그인 시스템은 `~/.claude/plugins/cache/<plugin>/` 경로에 플러그인 파일을 설치한다. 이 경로는 `/plugin install` 명령이 언제든지 완전히 덮어쓸 수 있는 배포 트리다. 따라서 그 하위에 사용자 데이터를 두는 것은 구조적으로 불안전하다.

이 불변식은 그 위험을 코드 레벨에서 차단한다. 플러그인 루트 내에서 허용되는 쓰기는 두 가지뿐이다: (1) 설치 게이트가 기록하는 버전 마커 파일, (2) `appendHookLog()`를 통한 `~/.local/state/sdi/hook.log` 단방향 기록. 그 외 모든 데이터 쓰기는 데몬(`sdid`)이 XDG 경로를 통해 수행한다.

`invariant.xdg-path-separation`과 쌍을 이루는 불변식이다. xdg-path-separation이 "어디에 데이터를 두어야 하는가"를 정의한다면, 이 불변식은 "어디에 두면 안 되는가"를 명시적으로 금지한다.

## 깨지면 무슨 일이 일어나나

플러그인 업데이트 또는 재설치 시 `~/.claude/plugins/cache/sdi/` 하위 트리 전체가 교체된다. 그 안에 SQLite DB가 있으면 모든 Plan·Scenario·Decision·Round·Task 이력이 소실된다. 소실은 경고 없이 일어나며, 회귀 검증의 기반인 시나리오 이력이 사라져 SDI의 핵심 가치인 "이전 시나리오 자동 재검증"이 불가능해진다. 로그가 플러그인 디렉터리에 있으면 bypass arm 남용 추적이 불가능해지며, 소켓이 거기 있으면 설치와 함께 소켓이 삭제되어 진행 중인 작업이 중단된다.

## 코드에서 어떻게 강제되나

데몬의 경로 해석 모듈(`crates/db/src/paths.rs`)은 XDG 경로 4개와 DB 파일 경로 총 5개를 확인하여 각각이 `~/.claude/plugins/` prefix 아래로 해석되는지 검사한다. 하나라도 겹치면 `sdid`가 시작 거부(startup panic)하며 사용자에게 stderr로 어떤 경로가 위반인지 출력한다. `sdi doctor`는 동일 검사를 비파괴적으로 재현하여 exit code 1을 반환한다. 플러그인 어댑터 코드(`plugin/adapters/shared/sdi-hooks.cjs`)는 `appendHookLog()` 외부에서 `fs.writeFile` / `fs.appendFile`을 호출하는 코드가 있으면 린트 테스트가 실패하도록 구조화되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(플러그인 디렉터리 배포 트리 비오염 원칙의 SDI 측 결정 엔트리) 연결 필요
