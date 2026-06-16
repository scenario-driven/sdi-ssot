---
id: endpoint.hook-pre-tool-use
kind: Endpoint
title: Hook `PreToolUse` (도구 실행 전 5단계 게이트 검사)
definition: "편집·실행 도구 호출 직전에 위임 규칙, 패턴 적합성, 활성 태스크 존재 여부, 리소스 클레임 충돌, 자율성 정책을 순서대로 검사하여 위반 시 차단하는 보호 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/pre-tool-use.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.task", type: "reads" }
  - { to: "concept.scenario", type: "reads" }
  - { to: "concept.autonomy-policy", type: "reads" }
  - { to: "concept.collaboration-pattern", type: "reads" }
  - { to: "domain.autonomy", type: "belongs-to" }
  - { to: "domain.collaboration-patterns", type: "supports" }
  - { to: "domain.plugin-runtime", type: "belongs-to" }
  - { to: "domain.governance-audit", type: "supports" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`PreToolUse` 훅은 Claude Code 에이전트가 `Edit`, `Write`, `MultiEdit`, `Bash`, `NotebookEdit`, `Agent`, `Task`, `TeamCreate`, `SendMessage` 도구를 호출하기 직전에 개입하여 5개의 게이트를 순차적으로 통과시키는 보호 장치다. 게이트 중 하나라도 위반이 감지되면 훅이 차단 결과를 반환하고 도구 호출이 실행되지 않는다.

이 훅은 SDI의 핵심 설계 결정 여러 개를 기계적으로 강제하는 단일 집행 지점이다. D21(위임 게이트), D26(패턴 무결성), D29(리소스 클레임), D14/D17/D18(자율성 정책)이 모두 이 훅을 통해 런타임에 적용된다. 규칙이 문서나 관례가 아니라 실행 경로 자체에 박혀 있기 때문에 에이전트가 의도치 않게 위반하는 것이 원칙적으로 불가능하다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

matcher 패턴 `Edit|Write|MultiEdit|Bash|NotebookEdit|Agent|Task|TeamCreate|SendMessage`에 해당하는 도구가 호출될 때마다 발화한다. 이 도구들은 파일 시스템 변경, 쉘 명령 실행, 에이전트 생성, 팀 메시지 발송 등 모두 외부 상태에 영향을 미칠 수 있는 도구들이다.

**우선순위 우회 경로**

정상 게이트 평가 전에 두 가지 우회 경로를 먼저 확인한다. 환경 변수 `SDI_BYPASS_HOOKS=1`이 설정되어 있으면 모든 게이트를 건너뛰고 도구 실행을 허용한다. 또는 `~/.cache/sdi/bypass-once` 파일이 존재하면 해당 발화에 한해 일회성 우회를 허용하고 파일을 삭제한다. 두 경우 모두 감사 로그에 우회 사실이 기록된다.

**Gate 1 — D21 위임 게이트**

주 세션(오케스트레이터)이 편집 도구(`Edit`, `Write`, `MultiEdit`, `NotebookEdit`)를 직접 호출하는 것을 차단한다. 훅 입력의 `agent_id` 필드가 없거나 비어 있으면 주 세션으로 간주하여 차단한다. 에이전트 전문가(전문화된 서브에이전트)만 편집 도구를 실행할 수 있다. `Bash`는 읽기 전용 명령 화이트리스트(`sdi`, `git status/log/diff`, `cat`, `grep`, `gh list/view` 등)에 해당하는 경우에만 주 세션에서 허용된다. 등록되지 않은 유형의 서브에이전트가 호출하면 차단이 아니라 경고(advisory)만 발행한다.

**Gate 2 — D26 패턴 어드바이저리 (비차단)**

`Agent`나 `Task` 도구 호출 시, 프롬프트 내용이 멀티에이전트 의도 신호(예: "specialist team", "parallel", "swarm", "fan-out", "agents-as-tools", "multi-agent" 등의 토큰 또는 `pattern_id` 필드)를 포함하고 있는데 활성 CollaborationPattern이 없으면 경고를 발행한다. 이 게이트는 차단하지 않으며, 에이전트가 올바른 패턴을 먼저 등록하도록 유도하는 어드바이저리 역할만 한다.

**Gate 3 — 활성 태스크 게이트**

`Edit`, `Write`, `MultiEdit`, `NotebookEdit` 도구 호출 시 진행 중인(`in_progress`) 태스크가 없으면 차단한다. 태스크 없이 파일을 편집하는 것은 SDI의 태스크 주도 개발 원칙을 위반하는 행위다. 에이전트는 편집을 시작하기 전에 반드시 태스크를 생성하고 활성화해야 한다.

**Gate 4 — D29 리소스 클레임 차단**

`Edit`, `Write`, `NotebookEdit` 도구가 접근하려는 대상 파일 경로에 대해 `GET /scenarios/active-claims`를 조회한다. 다른 시나리오가 동일 경로 또는 해당 경로를 커버하는 글로브 패턴으로 클레임을 보유 중이면 종료 코드 2로 차단한다. 차단 시 구조화된 JSON 오류 페이로드(`{ block: 'sdi_claim_overlap', target_path, my_scenario, holders, hint }`)를 출력하여 에이전트가 충돌 원인을 파악할 수 있게 한다. 데몬에 접근할 수 없는 경우에는 안전하게 통과를 허용하여 데몬 장애가 편집 작업 전체를 막지 않도록 한다.

**Gate 5 — D14/D17/D18 자율성 게이트**

활성 AutonomyPolicy가 L3 이하인 경우, 도구 차단 대신 `permissionDecision: 'ask'`를 응답에 포함하여 사용자에게 확인을 요청한다. L4/L5 조건에서는 정책에 따라 자동으로 허용하거나 추가 검증을 거친다.

**차단 응답 형식**

차단이 결정되면 훅은 `{ hookSpecificOutput: { permissionDecision: 'deny', permissionDecisionReason: '<이유>' } }` 형태의 JSON을 출력하고 종료한다. Claude Code는 이를 수신해 도구 호출을 중단한다.

## 권한 / 제약

이 훅은 다섯 개의 게이트를 순서대로 평가하며, 앞선 게이트에서 차단이 결정되면 이후 게이트는 평가하지 않는다. Gate 2는 유일하게 비차단 어드바이저리 게이트다.

`SDI_BYPASS_HOOKS=1` 또는 `~/.cache/sdi/bypass-once`를 통한 우회는 모두 감사 로그에 남는다. 우회는 긴급 상황 전용이며, 일상적 우회는 프로토콜 위반으로 간주된다. `sdi bypass arm --reason "<이유>"` 명령이 공식 일회성 우회 진입점이다.

데몬이 응답하지 않으면 Gate 4(클레임 충돌 검사)만 통과를 허용하고 나머지 게이트는 정상 평가한다. 이는 데몬 장애로 인해 편집 작업 전체가 마비되는 상황을 방지하기 위한 의도적 설계다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: `Edit|Write|MultiEdit|Bash|NotebookEdit|Agent|Task|TeamCreate|SendMessage`
- 사용하는 데몬 엔드포인트: `GET /projects/by-cwd`, `GET /tasks/in-flight`, `GET /patterns/active`, `GET /scenarios/active-claims`
- 우회 경로: `SDI_BYPASS_HOOKS=1` 환경 변수, `~/.cache/sdi/bypass-once` 파일
- 관련 설계 결정: D13 (멀티에이전트 우선), D14 (자율성 정책), D17 (모드 기본값), D18 (서킷 브레이커), D20 (합의/불일치 단위 자율성 게이트), D21 (위임 게이트), D25 (패턴 범위별 자율성), D26 (4-패턴 무결성 게이트), D29 (멀티세션 리소스 클레임)

## 미확정 (OPEN)

- Bash 읽기 전용 화이트리스트의 정확한 패턴 목록이 코드 확인 전까지 미확정
- Gate 1에서 "등록되지 않은 유형의 서브에이전트"를 판별하는 기준이 명확하지 않음
- Gate 2의 멀티에이전트 의도 신호 토큰 목록이 고정인지 확장 가능한지 미확인
- Gate 5에서 L3/L4/L5 경계별 정확한 허용/요청/차단 동작 분기 미확인
- 다섯 게이트 중 어디서 차단이 발생했는지 차단 이유에 게이트 번호가 포함되는지 미확인
- `bypass-once` 파일의 TTL 기본값이 60초로 고정인지, 설정 가능한지 미확인
