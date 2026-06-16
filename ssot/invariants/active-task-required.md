---
id: invariant.active-task-required
kind: Invariant
title: 변경 작업에는 활성 태스크가 반드시 있어야 한다
definition: "등록·활성화된 프로젝트 안에서 코드를 바꾸는 모든 행위는, 현재 진행 중(in_progress)인 태스크가 지정되어 있을 때에만 허용된다. 활성 태스크가 없으면 변경 도구 호출이 시작 자체부터 막힌다."
governs:
  - concept.task
  - domain.task-decomposition
  - domain.plugin-runtime
  - domain.governance-audit
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/pre-tool-use.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI가 관리하는 프로젝트 안에서는, "지금 무슨 일을 하고 있는가"를 가리키는 진행 중 태스크가 반드시 먼저 지정되어 있어야만 코드를 바꿀 수 있다. 파일을 고치거나 새로 쓰는 일, 임의의 쉘 명령 실행, 서브에이전트 스폰, 팀 생성, 에이전트 간 메시지 전송 같은 "변경을 일으키는 행위" 전부가 이 규칙의 대상이다.

활성 태스크가 없으면 이러한 행위는 PreToolUse 훅에서 시작 단계에 차단되며, 차단 메시지에는 `/scenario new`와 `/round start`를 통한 자가 복구 경로가 안내된다.

이 불변식은 SDI의 핵심 흐름을 강제한다. Task는 LLM이 시나리오에서 분해하는 런타임 산출물이므로, 태스크가 없다는 것은 어떤 시나리오도 현재 진행 중이지 않다는 의미다. 태스크 없이 코드를 바꾸면 그 변경이 어떤 시나리오 계약을 이행하기 위한 것인지 추적할 수 없고, `PostToolUse` 훅의 자동 증거 수집도 의미를 잃는다.

프로젝트가 `enabled=false`로 설정되거나 미등록 cwd에서 작업하는 경우 이 검사는 무동작(`project-disabled` 감사 기록)이 된다. 비상 우회는 `sdi bypass arm --reason "<사유>"` 명령으로만 허용되며, 우회 자체가 감사 로그에 기록된다.

## 깨지면 무슨 일이 일어나나

활성 태스크 없이 코드가 변경되면 어떤 시나리오를 위한 변경인지 알 수 없다. `PostToolUse` 훅이 변경 파일을 Task evidence에 기록하려 해도 연결할 태스크가 없어 증거가 사라진다. 이는 Round의 scenario_results 집계에서 어떤 시나리오가 어떤 변경으로 passing이 됐는지 확인할 방법이 없어지는 것이다. R2+ 회귀 검증에서 근거 없이 passing으로 표기된 시나리오가 생길 수 있으며, 감사 추적이 없는 변경이 누적되면 "이 코드가 왜 이 모양인가"를 재구성할 수 없게 된다.

## 코드에서 어떻게 강제되나

Claude Code의 PreToolUse 이벤트에 훅이 걸려 있다. `plugin/hooks/hooks.json`의 매처가 변경을 일으키는 도구군(`Edit`, `Write`, `MultiEdit`, `Bash`, `NotebookEdit`, `Agent`, `Task`, `TeamCreate`, `SendMessage`)을 대상으로 하며, `plugin/adapters/shared/sdi-hooks.cjs`의 훅 바디가 현재 cwd의 프로젝트에 in_progress 태스크가 있는지 데몬에 질의한다. 없으면 `permissionDecision: deny` 페이로드를 반환하여 도구 실행이 거부된다. HOOK_ENFORCEMENT.md의 신뢰 경계 #1로도 명시되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(활성 태스크 없이 변경 불가 정책의 SDI 측 결정 엔트리) 연결 필요
