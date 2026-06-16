---
id: endpoint.hook-post-tool-use
kind: Endpoint
title: Hook `PostToolUse` (파일 편집 완료 후 태스크 활동 기록)
definition: "편집 도구가 성공적으로 완료된 직후 어떤 파일이 어떤 도구로 변경되었는지를 활성 태스크의 활동 이력에 기록하는 감사 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/post-tool-use.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.task", type: "mutates" }
  - { to: "concept.task-evidence", type: "supports" }
  - { to: "concept.project", type: "reads" }
  - { to: "domain.governance-audit", type: "belongs-to" }
  - { to: "domain.plugin-runtime", type: "belongs-to" }
  - { to: "domain.task-decomposition", type: "supports" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`PostToolUse` 훅은 `Edit`, `Write`, `MultiEdit`, `NotebookEdit` 도구가 성공적으로 실행된 직후 발화하는 감사 훅이다. 이 훅의 유일한 역할은 "어떤 태스크 진행 중에 어떤 도구가 어떤 파일을 변경했는가"를 데몬에 기록하는 것이다.

이 기록은 SDI의 근거 추적 원칙을 뒷받침한다. 태스크가 완료되었을 때, 그 태스크 동안 어떤 파일들이 수정되었는지 활동 이력으로 남아 있어야 한다. 이를 통해 에이전트는 나중에 특정 태스크가 어떤 파일에 영향을 미쳤는지 추적할 수 있고, 회귀 검증이나 혼선 감지 시 참조 자료로 활용된다.

이 훅은 컨텍스트를 주입하거나 차단 결정을 내리지 않는다. 오직 기록만 담당한다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

matcher 패턴 `Edit|Write|MultiEdit|NotebookEdit`에 해당하는 도구가 성공적으로 완료된 후에 발화한다. PreToolUse 훅과 달리 도구 실행 이후에 발화하므로, 이미 파일 변경이 일어난 상태에서 그 사실을 기록하는 시점이다.

**선결 조건**

환경 변수 `SDI_ACTIVE_TASK`가 설정되어 있지 않으면 훅은 아무 동작 없이 조용히 종료한다. 태스크 ID 없이는 어느 태스크의 활동으로 기록해야 할지 알 수 없으므로, 이 경우 기록을 시도하지 않는다. 이 동작은 PreToolUse 훅의 Gate 3(활성 태스크 게이트)과 짝을 이룬다. PreToolUse 훅이 태스크 없이 편집을 차단했다면 PostToolUse 훅이 발화할 일이 없지만, 우회 경로를 통해 편집이 이루어진 경우에도 태스크가 없으면 이 훅은 조용히 무시한다.

**기록 흐름**

1. `GET /projects/by-cwd`로 현재 작업 디렉토리에 연결된 프로젝트 ID를 조회한다.
2. `POST /activity`로 활동 항목을 데몬에 전송한다. 전송하는 페이로드에는 프로젝트 ID, 활동 종류(`task.file_touched`), 요약 문자열(`<도구명> <파일경로>`), 태스크 ID(`entity_id`), 그리고 세부 정보(`{ tool, file }`)가 포함된다.

**출력 없음**

이 훅은 Claude Code의 세션 컨텍스트에 아무것도 주입하지 않는다. 에이전트가 인지하지 못하는 사이에 백그라운드로 기록만 이루어지는 레코드 전용 훅이다. 차단 결정도 없고 경고 주입도 없다.

## 권한 / 제약

이 훅은 `POST /activity` 하나의 쓰기 요청만 수행한다. 태스크 상태를 변경하거나 파일을 수정하는 부작용은 없다. 기록 실패가 파일 변경 작업의 유효성에 영향을 주지 않는다. 즉, 데몬이 응답하지 않아 기록에 실패하더라도 이미 완료된 파일 편집은 그대로 유지된다.

기록되는 파일 경로는 도구가 실제로 편집한 경로 그대로이며, 훅이 경로를 가공하거나 익명화하지 않는다.

`SDI_BYPASS_HOOKS=1` 환경 변수가 설정된 경우 이 훅을 포함한 모든 훅이 우회된다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: `Edit|Write|MultiEdit|NotebookEdit`
- 사용하는 데몬 엔드포인트: `GET /projects/by-cwd`, `POST /activity`
- 전송하는 활동 종류: `task.file_touched`
- 선결 조건: `SDI_ACTIVE_TASK` 환경 변수 필수
- 관련 설계 결정: D12 (스냅샷 전용 문서), D21 (위임 게이트 — PreToolUse 짝), D29 (멀티세션 리소스 클레임)

## 미확정 (OPEN)

- `SDI_ACTIVE_TASK` 미설정 시 조용한 종료가 감사 로그에 남는지, 완전 무시인지 미확인
- 데몬 `POST /activity` 호출 실패 시 재시도 로직 존재 여부 미확인
- `MultiEdit`의 경우 여러 파일이 한 번에 변경될 때 활동 항목이 파일당 하나씩 생성되는지, 하나로 묶이는지 미확인
- 요약 문자열 `<도구명> <파일경로>`의 정확한 포맷 및 파일 경로가 절대 경로인지 상대 경로인지 미확인
- 프로젝트가 `enabled=false`인 경우 이 훅이 어떻게 동작하는지 미확인
