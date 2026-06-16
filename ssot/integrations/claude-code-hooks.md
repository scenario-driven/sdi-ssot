---
id: integration.claude-code-hooks
kind: Integration
title: Claude Code 훅 연동
purpose: Claude Code가 도구를 실행하거나 세션을 시작하는 매 시점에 SDI의 가드레일을 끼워 넣는다 — 활성 태스크 없이 코드 변경 차단, 오케스트레이터의 실행 도구 직접 호출 차단(D21 Delegation Gate), 패턴 무결성 경고(D26), 리소스 클레임 충돌 차단(D29), 세션 컨텍스트 자동 주입.
definition: Claude Code 훅 이벤트(SessionStart·UserPromptSubmit·PreToolUse·PostToolUse·SubagentStart·SubagentStop)를 Node.js 핸들러로 받아, SDI 데몬의 HTTP API와 대화하며 게이트·기록·컨텍스트 주입을 수행하는 연동.
integratesWith: []
implementedIn:
  - sdi-plugin/plugin/hooks/hooks.json
  - sdi-plugin/plugin/adapters/claude/session-start.cjs
  - sdi-plugin/plugin/adapters/claude/user-prompt-submit.cjs
  - sdi-plugin/plugin/adapters/claude/pre-tool-use.cjs
  - sdi-plugin/plugin/adapters/claude/post-tool-use.cjs
  - sdi-plugin/plugin/adapters/claude/subagent-start.cjs
  - sdi-plugin/plugin/adapters/claude/subagent-stop.cjs
  - sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs
impacts:
  - domain.plugin-runtime
  - domain.agent-coordination
  - domain.autonomy
  - domain.governance-audit
relatesTo:
  - to: domain.plugin-runtime
    type: realizes
    note: 훅 등록이 곧 플러그인 런타임 진입점이다
  - to: domain.governance-audit
    type: realizes
    note: 모든 훅 호출은 audit 로그에 기록되어 거버넌스 감사 근거가 된다
  - to: domain.agent-coordination
    type: realizes
    note: SubagentStart/Stop 훅이 에이전트 실행을 태스크에 바인딩한다
  - to: domain.autonomy
    type: realizes
    note: D21 Delegation Gate와 D26 패턴 무결성 검사가 AutonomyPolicy 시행의 실행 면이다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 Claude Code 그 자체다. Claude Code는 사용자와 코딩 에이전트가 작업하는 동안 정해진 시점마다 훅 이벤트를 발화한다. SDI는 이 이벤트들을 Node.js 핸들러로 받아 SDI의 가드레일과 컨텍스트 주입을 끼워 넣는다.

훅 이벤트는 여섯 종류다.

- **SessionStart**: 세션이 시작·초기화·컴팩션될 때 발화한다. 설치 게이트(install gate)를 통해 `sdi`·`sdid` 바이너리와 스킬 파일을 확인하고, 데몬을 기동한 뒤 활성 프로젝트의 작업 요약과 규칙을 컨텍스트로 주입한다.
- **UserPromptSubmit**: 사용자가 프롬프트를 제출할 때마다 발화한다. 현재 활성 시나리오의 컨텍스트를 주입하고, 활성 태스크가 없으면 경고한다.
- **PreToolUse**: 도구를 실행하기 직전에 발화한다. 대상 도구(`Edit`, `Write`, `MultiEdit`, `Bash`, `NotebookEdit`, `Agent`, `Task`, `TeamCreate`, `SendMessage`)에 대해 네 단계 게이트를 순서대로 통과시킨다: 활성 태스크 확인 → D21 Delegation(오케스트레이터의 실행 도구 차단) → D26 패턴 무결성 경고 → D29 리소스 클레임 충돌 차단. 게이트가 차단하면 종료 코드 2와 구조화된 JSON 페이로드를 반환한다.
- **PostToolUse**: 파일 편집(`Edit`, `Write`, `MultiEdit`, `NotebookEdit`) 직후 발화한다. 변경 파일을 활성 태스크 증거로 자동 기록한다.
- **SubagentStart**: 서브에이전트가 시작될 때 발화한다. 서브에이전트 실행을 활성 시나리오·태스크에 바인딩한다.
- **SubagentStop**: 서브에이전트가 종료될 때 발화한다. 실행 결과 요약을 태스크 증거로 기록하고, 조건이 충족되면 태스크를 자동 완료 처리한다.

프로토콜은 Claude Code 공식 훅 규약을 그대로 따른다. 각 이벤트는 명령으로 Node.js 스크립트를 실행하고, 핸들러가 표준 출력으로 돌려주는 결정을 Claude Code가 해석한다. 인증이나 외부 네트워크 호출은 없으며, 모든 판단은 로컬 데몬 HTTP API(`127.0.0.1:<port>`)를 통해서만 이뤄진다.

## 구현 위치 (provenance)

어느 이벤트에 어떤 핸들러를 걸지는 훅 매니페스트(`plugin/hooks/hooks.json`)가 선언한다. 이 파일은 이벤트 이름·매처(정규식)·실행 명령을 기술하며, 핸들러 경로는 `${CLAUDE_PLUGIN_ROOT}` 환경 변수를 참조한다.

각 이벤트의 진입점은 `plugin/adapters/claude/` 아래의 얇은 `.cjs` 파일들이다. 이 파일들은 2줄짜리 shim으로 실제 모든 로직을 공유 핸들러(`plugin/adapters/shared/sdi-hooks.cjs`)에 위임한다. 공유 핸들러가 바이너리 탐색·설치 게이트·데몬 HTTP 통신·게이트 판정·컨텍스트 주입·감사 로그 기록의 단일 집결지다.

비상 우회는 `sdi bypass arm --reason "<사유>"` 명령으로 XDG 캐시에 JSON 마커를 써서 한 번의 도구 호출에 대해 D21·D29 게이트를 해제하고 자동 소비된다. `SDI_BYPASS_HOOKS=1`(전체 PreToolUse 우회), `SDI_DELEGATION_BYPASS=1`(D21만), `SDI_HOOK_V05_DISABLE=1`(D26+D29 비활성화) 환경 변수는 Claude Code를 해당 셸에서 새로 띄울 때만 유효하다.

## 불변식

- 메인 세션(오케스트레이터)은 실행 도구(`Edit`/`Write`/`MultiEdit`/`NotebookEdit`/mutating `Bash`)를 직접 호출할 수 없다. PreToolUse가 `hookInput.agent_id` 부재를 감지하면 즉시 차단한다(D21). 이 게이트가 깨지면 SDI의 "multi-agent orchestration이 본체"(D13) 원칙이 무너진다.
- 리소스 클레임이 겹치는 다른 시나리오가 존재하면 해당 파일 수정이 차단된다(D29). 데몬이 내려가면 이 게이트는 통과시킨다(편집기 락 방지 우선).
- 모든 훅 호출과 우회 시도는 `~/.local/state/sdi/hook.log`에 감사 로그로 남는다.
- 설치 게이트는 멱등이다. 바이너리·스킬 파일·데몬 헬스가 모두 확인되면 무동작으로 통과한다.

## 영향 범위

이 연동이 끊기거나 잘못 등록되면 여섯 훅 이벤트 전부가 작동하지 않는다. 그 결과 코딩 에이전트에 대한 가드레일(활성 태스크 강제·D21 위임 게이트·D26 패턴 경고·D29 클레임 차단)이 사라지고, 세션 컨텍스트 자동 주입과 서브에이전트 결과 기록도 중단된다. 실질적으로 SDI가 Claude Code 위에서 동작하는 방식 전체가 이 연동에 달려 있다.

## 미확정 (OPEN)
- [ ] OPEN: 각 게이트의 타임아웃 정책(훅 응답 지연 시 차단 vs 통과 여부)이 이벤트별로 어떻게 다른지 명시적 문서화 필요.
- [ ] OPEN: D29 클레임 겹침 사용자 프롬프트("merge or wait")의 정확한 UX 흐름과 사용자 응답 처리 경로 확인 필요.
