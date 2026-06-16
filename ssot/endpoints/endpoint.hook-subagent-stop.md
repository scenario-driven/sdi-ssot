---
id: endpoint.hook-subagent-stop
kind: Endpoint
title: Hook `SubagentStop` (서브에이전트 종료 시 결과 요약 기록)
definition: "서브에이전트가 종료될 때 에이전트 이름과 작업 결과 요약을 활성 태스크의 활동 이력에 기록하는 감사 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/subagent-stop.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.task", type: "mutates" }
  - { to: "concept.run", type: "supports" }
  - { to: "concept.agent-note", type: "supports" }
  - { to: "concept.collaboration-pattern", type: "supports" }
  - { to: "concept.task-evidence", type: "supports" }
  - { to: "domain.agent-coordination", type: "belongs-to" }
  - { to: "domain.governance-audit", type: "supports" }
  - { to: "domain.plugin-runtime", type: "belongs-to" }
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`SubagentStop` 훅은 서브에이전트가 작업을 마치고 종료될 때 자동으로 발화하는 감사 훅이다. matcher 없이 등록되어 있어 어떤 종류의 서브에이전트가 종료되어도 예외 없이 발화한다.

`SubagentStart` 훅이 서브에이전트 기동 시점을 기록하는 것과 대칭적으로, 이 훅은 서브에이전트가 종료될 때 그 결과 요약을 태스크 활동 이력에 남긴다. 두 훅이 함께 동작하면 특정 서브에이전트가 어떤 태스크의 맥락에서 기동되어 어떤 결과를 냈는지 완전한 구간 기록이 형성된다.

이 기록은 멀티에이전트 협업 흐름에서 각 전문 에이전트가 무엇을 했는지 사후에 추적하거나, 의도하지 않은 결과가 발생했을 때 어느 에이전트의 어떤 단계에서 문제가 생겼는지 파악하는 데 활용된다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

matcher가 설정되지 않아 어떤 서브에이전트가 종료되어도 발화한다. 서브에이전트가 정상 종료하든, 오류로 종료하든 모두 해당된다.

**선결 조건**

환경 변수 `SDI_ACTIVE_TASK`가 설정되어 있지 않으면 훅은 아무 동작 없이 조용히 종료한다. `SubagentStart`와 동일한 선결 조건이며, 동일한 이유다. 태스크 컨텍스트 없이 결과를 기록할 귀속 대상이 없기 때문이다.

**기록 흐름**

1. `GET /projects/by-cwd`로 현재 작업 디렉토리에 연결된 프로젝트 ID를 조회한다.
2. `POST /activity`로 서브에이전트 종료 활동을 데몬에 전송한다. 전송하는 페이로드에는 프로젝트 ID, 활동 종류(`subagent.stop`), 에이전트 이름(네임스페이스 접두사가 제거된 단순 이름), 태스크 ID(`entity_id`), 그리고 서브에이전트가 반환한 결과 텍스트의 앞 200자가 포함된다.

결과 텍스트는 200자로 잘린다. 이는 전체 결과를 저장하려는 의도가 아니라, 어떤 결과가 나왔는지 나중에 대략 파악할 수 있도록 하는 요약 미리보기 역할이다. 전체 결과는 에이전트가 오케스트레이터에게 반환하는 정상 채널을 통해 전달되므로, 이 훅의 기록은 참조 목적으로만 사용된다.

에이전트 이름의 네임스페이스 접두사는 `SubagentStart`와 동일하게 제거된다.

**출력 없음**

이 훅도 Claude Code의 세션 컨텍스트에 아무것도 주입하지 않는다. 완전히 백그라운드에서 동작하는 레코드 전용 훅이다.

## 권한 / 제약

이 훅은 `POST /activity` 하나의 쓰기 요청만 수행한다. 서브에이전트의 종료 처리나 결과 반환에 영향을 주지 않으며, 기록 실패가 서브에이전트의 종료 완료를 방해하지 않는다.

결과 텍스트의 200자 제한은 고정값으로 추론되며, 이 제한이 설정 가능한지 여부는 코드 확인 전까지 미확정이다.

`SDI_BYPASS_HOOKS=1` 환경 변수가 설정된 경우 이 훅을 포함한 모든 훅이 우회된다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: 없음(모든 서브에이전트 종료에 적용)
- 사용하는 데몬 엔드포인트: `GET /projects/by-cwd`, `POST /activity`
- 전송하는 활동 종류: `subagent.stop`
- 선결 조건: `SDI_ACTIVE_TASK` 환경 변수 필수
- 짝이 되는 훅: `endpoint.hook-subagent-start`
- 관련 설계 결정: D13 (멀티에이전트 오케스트레이션 우선), D15 (4가지 내장 멀티에이전트 패턴), D21 (위임 게이트), D22 (CollaborationPattern 엔티티)

## 미확정 (OPEN)

- 결과 텍스트 200자 제한이 설정 가능한지, 고정값인지 미확인
- 서브에이전트가 오류로 종료될 때 결과 텍스트로 어떤 내용이 전달되는지 미확인 (에러 메시지인지 빈 문자열인지)
- `SDI_ACTIVE_TASK` 미설정 시 조용한 종료가 감사 로그에 남는지 완전 무시인지 미확인
- 데몬 `POST /activity` 호출 실패 시 재시도 로직 존재 여부 미확인
- `SubagentStart`와 `SubagentStop` 간 매칭을 추적하는 단위 식별자(예: 에이전트 실행 ID)가 페이로드에 포함되는지 미확인 — 없으면 같은 에이전트가 여러 번 기동될 때 시작/종료 쌍 구분이 어려울 수 있음
