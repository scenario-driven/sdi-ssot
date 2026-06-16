---
id: component.hooks-adapter
kind: SystemComponent
title: SDI 훅 어댑터 (Claude Code 이벤트 연결 레이어)
purpose: Claude Code의 생명주기 훅(SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, SubagentStart, SubagentStop)을 가로채 SDI 불변식을 집행하는 CJS 스크립트 집합이다. 활성 태스크 없는 뮤테이션 차단, 위임 게이트(D21), 패턴 형상 경고(D26), 리소스 클레임 충돌 차단(D29), 파일 변경 자동 기록이 여기서 이뤄진다.
realizedBy:
  - domain.governance-audit
  - domain.plugin-runtime
  - domain.agent-coordination
  - domain.collaboration-patterns
implementedIn:
  - sdi-plugin/plugin/adapters
  - sdi-plugin/plugin/hooks
dependsOn:
  - component.daemon
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
  - persona.subagent
  - persona.solo-builder
relatesTo:
  - to: component.plugin-shell
    type: belongs-to
    note: plugin/adapters/ 와 plugin/hooks/hooks.json 이 플러그인 셸 안에 있다
  - to: component.daemon
    type: depends-on
    note: 훅 스크립트가 데몬 HTTP API를 호출해 상태를 읽고 게이트 결정을 내린다
  - to: domain.governance-audit
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

훅 어댑터는 Claude Code가 발화하는 생명주기 이벤트마다 SDI 규칙을 적용하는 Node.js CJS 스크립트 레이어다. `plugin/hooks/hooks.json`이 어떤 이벤트에 어떤 스크립트를 연결할지 선언하고, `plugin/adapters/claude/` 아래 각 훅 스크립트가 실제 로직을 수행한다.

각 훅의 역할은 다음과 같다.

**SessionStart**: 세션 시작 시 SDI 대시보드 컨텍스트를 Claude Code 컨텍스트에 주입한다. 활성 태스크·라운드 상태를 에이전트에게 미리 알린다.

**UserPromptSubmit**: 사용자 입력마다 현재 활성 태스크 컨텍스트를 주입한다. 활성 태스크가 없으면 경고를 표시해 에이전트가 태스크를 먼저 설정하도록 유도한다.

**PreToolUse**: 가장 많은 게이트가 집행되는 훅이다. Edit·Write·MultiEdit·NotebookEdit·Bash에 대해서는 활성 태스크 존재 여부를 확인하고, 없으면 뮤테이션을 차단한다(exit code 2). D21 위임 게이트는 메인 세션의 실행 도구 직접 호출을 막고 서브에이전트만 호출 가능하게 한다. D26 패턴 형상 경고는 Agent·Task·TeamCreate 도구에 다중 에이전트 의도 토큰이 포함될 때 활성 패턴 row가 없으면 경고를 내보낸다(비차단). D29 리소스 클레임 충돌은 Edit·Write 대상 경로가 다른 시나리오의 클레임과 겹치면 차단한다. bypass가 무장된 경우 단 한 번의 도구 호출에 한해 모든 게이트를 우회한다.

**PostToolUse**: Edit·Write·MultiEdit·NotebookEdit 완료 후 변경된 파일 경로를 활성 태스크에 자동 기록한다. 태스크 증거 추적의 자동화 부분이다.

**SubagentStart/SubagentStop**: 서브에이전트 시작 시 태스크에 바인딩하고, 종료 시 결과 요약을 태스크에 추가한다.

## 경계와 의존

훅 어댑터는 반드시 데몬이 동작 중인 상태에서 의미 있게 작동한다. 데몬이 내려가 있으면 상태를 읽을 수 없으므로, 대부분의 게이트는 데몬 미가동 시 우아하게 통과시킨다(proceed 정책). D29 클레임 충돌은 특히 "데몬 미가동 → 차단 없음"을 명시한다. 훅 스크립트는 `plugin/adapters/shared/sdi-hooks.cjs`에 공유 유틸리티를 둬 각 어댑터가 재사용한다.

## 통신 패턴

Node.js CJS 스크립트가 데몬의 HTTP REST API를 동기적으로 호출(fetch/axios 계열)한다. 훅 이벤트는 Claude Code 런타임이 동기적으로 처리하므로 응답이 빠르지 않으면 에이전트 응답이 지연된다. exit code 2를 반환하면 Claude Code가 해당 도구 호출을 차단한다.

## 하위 서브패키지 (책임 단위)

- **hooks.json**: 이벤트 → 스크립트 매핑 선언.
- **adapters/claude/session-start.cjs**: 세션 시작 컨텍스트 주입.
- **adapters/claude/user-prompt-submit.cjs**: 활성 태스크 컨텍스트 주입 + 무태스크 경고.
- **adapters/claude/pre-tool-use.cjs**: D21 위임 게이트, 활성 태스크 차단, D26 패턴 경고, D29 클레임 충돌 차단, bypass 게이트.
- **adapters/claude/post-tool-use.cjs**: 파일 변경 자동 기록.
- **adapters/claude/subagent-start.cjs** / **subagent-stop.cjs**: 서브에이전트 태스크 바인딩과 결과 기록.
- **adapters/shared/sdi-hooks.cjs**: 공유 유틸리티(데몬 연결, 활성 태스크 조회 등).

## 미확정 (OPEN)

- [ ] OPEN: bypass 무장 상태의 TTL(기본 60초) 과 XDG 캐시 파일 구조 확인 필요.
- [ ] OPEN: SDI_HOOK_V05_DISABLE=1 환경 변수가 D26 + D29 게이트만 끄는지 전체 훅을 끄는지 코드에서 확인 필요.
