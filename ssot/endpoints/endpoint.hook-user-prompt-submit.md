---
id: endpoint.hook-user-prompt-submit
kind: Endpoint
title: Hook `UserPromptSubmit` (프롬프트 제출 시 활성 태스크 컨텍스트 주입)
definition: "사용자가 프롬프트를 제출할 때마다 현재 활성 태스크와 플랜 컨텍스트를 주입하고, 태스크가 없을 경우 작업 분해를 촉구하는 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/user-prompt-submit.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.task", type: "reads" }
  - { to: "concept.plan", type: "reads" }
  - { to: "concept.project", type: "reads" }
  - { to: "concept.dispatch", type: "supports" }
  - { to: "domain.task-decomposition", type: "supports" }
  - { to: "domain.plugin-runtime", type: "belongs-to" }
  - { to: "domain.planning", type: "supports" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`UserPromptSubmit` 훅은 사용자가 Claude Code 채팅창에 메시지를 입력하고 제출할 때마다 발화하는 훅이다. matcher 없이 등록되어 있어 모든 프롬프트 제출 이벤트에 적용된다. 이 훅의 목적은 에이전트가 매번 새 메시지를 처리하기 전에 "어떤 프로젝트, 어떤 플랜, 어떤 태스크 맥락 위에서 작동하는가"를 명확히 인지하게 만드는 것이다.

SDI의 태스크 주도 개발 원칙에 따르면, 에이전트는 반드시 진행 중인 태스크 단위로 작업해야 한다. 이 훅은 그 규율을 매 프롬프트 제출 시점에 기계적으로 강화한다. 에이전트가 태스크 없이 편집 작업을 시도하면 PreToolUse 훅이 차단하지만, UserPromptSubmit 훅은 그보다 앞서 "지금 어떤 태스크인가"를 상기시켜 무의식적인 이탈을 예방한다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

matcher가 설정되지 않아 사용자가 프롬프트를 제출할 때마다 예외 없이 발화한다. `/clear`나 세션 재시작 이후에도 계속 발화한다.

**데이터 수집 흐름**

훅 스크립트는 발화할 때마다 다음 순서로 데몬에 질의한다.

1. `GET /projects/by-cwd`로 현재 작업 디렉토리에 연결된 프로젝트를 조회한다.
2. `GET /plans/active`로 해당 프로젝트의 활성 플랜을 조회한다.
3. `GET /tasks/in-flight`로 현재 진행 중인 태스크 목록을 조회한다.
4. 환경 변수 `SDI_ACTIVE_TASK`가 설정되어 있으면 해당 태스크 ID를 사용해 특정 태스크의 상세 정보를 별도로 가져온다.

**주입 결과 — 세 가지 경우**

수집 결과에 따라 주입 내용이 세 경우로 나뉜다.

첫 번째, `SDI_ACTIVE_TASK`가 설정된 경우다. 고정된 태스크 ID, 제목, 현재 상태가 컨텍스트 블록에 포함되어 에이전트가 "지금 이 태스크를 작업 중이다"라는 사실을 인지한 상태에서 프롬프트를 해석하게 된다.

두 번째, `SDI_ACTIVE_TASK`가 없지만 진행 중인 태스크가 존재하는 경우다. 진행 중인 태스크를 최대 5건까지 목록으로 주입하고, "고정된 태스크 없음" 경고 메시지를 함께 포함한다. 에이전트는 이 경고를 통해 작업을 시작하기 전에 태스크를 명시적으로 고정할 것을 인지하게 된다.

세 번째, 진행 중인 태스크가 하나도 없는 경우다. 컨텍스트에 "진행 중인 태스크 없음" 메시지와 함께 현재 시나리오를 태스크로 분해하라는 지시가 포함된다. 이는 에이전트가 빈 태스크 상태에서 편집을 시도하기 전에 먼저 분해 과정을 거치도록 유도하기 위함이다.

세 경우 모두 공통으로 프로젝트 키와 ID, 활성 플랜 제목과 ID가 컨텍스트 블록 앞부분에 포함된다.

**감사 로그**

발화할 때마다 `~/.local/state/sdi/hook.log`에 `user_prompt_submit` 이벤트 유형으로 감사 항목이 기록된다.

**차단 없음**

이 훅 자체는 프롬프트를 차단하거나 거부하지 않는다. 순수하게 컨텍스트를 주입하는 역할만 한다. 실제 편집 작업의 차단은 PreToolUse 훅이 담당한다.

## 권한 / 제약

이 훅은 읽기 전용이다. 데몬에 쓰기 요청을 보내지 않으며, 세션 상태를 변경하지 않는다. 주입하는 컨텍스트는 에이전트의 메시지 해석에 영향을 줄 뿐 Claude Code의 실행 흐름을 직접 제어하지 않는다.

모든 프롬프트 제출마다 발화하므로 데몬과 통신이 빈번히 발생한다. 데몬이 응답하지 않거나 프로젝트가 미등록 상태인 경우의 처리 방식은 코드 확인 전까지 미확정이다.

`SDI_BYPASS_HOOKS=1` 환경 변수가 설정된 경우 이 훅을 포함한 모든 훅이 우회된다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: 없음(모든 프롬프트에 적용)
- 사용하는 데몬 엔드포인트: `GET /projects/by-cwd`, `GET /plans/active`, `GET /tasks/in-flight`, 특정 태스크 상세 조회(선택적)
- 감사 로그 경로: `~/.local/state/sdi/hook.log`
- 관련 설계 결정: D3 (태스크는 런타임 아티팩트), D13 (멀티에이전트 우선), D16 (기본 = 정책에 따라 행동)

## 미확정 (OPEN)

- 진행 중인 태스크 최대 5건이라는 상한이 하드코딩인지 설정 가능한지 미확인
- `SDI_ACTIVE_TASK` 미설정 상태에서 특정 태스크 상세 조회가 생략되는 로직의 정확한 분기 조건 미확인
- 데몬 미응답 또는 프로젝트 미등록 시 훅이 조용히 무시되는지, 오류 메시지를 주입하는지 미확인
- 프로젝트가 `enabled=false`인 경우 이 훅이 어떻게 동작하는지 미확인
- 주입되는 컨텍스트 블록의 정확한 필드 구조와 최대 크기 제한 미확인
