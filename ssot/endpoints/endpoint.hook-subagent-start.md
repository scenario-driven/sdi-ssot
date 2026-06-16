---
id: endpoint.hook-subagent-start
kind: Endpoint
title: Hook `SubagentStart` (서브에이전트 시작 시 태스크 바인딩 기록)
definition: "서브에이전트가 시작될 때 해당 에이전트를 현재 활성 태스크에 연결하는 활동을 데몬에 기록하는 감사 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/subagent-start.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.task", type: "mutates" }
  - { to: "concept.run", type: "supports" }
  - { to: "concept.agent-note", type: "supports" }
  - { to: "concept.collaboration-pattern", type: "supports" }
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

`SubagentStart` 훅은 Claude Code 내에서 서브에이전트(전문화된 하위 에이전트)가 기동될 때 자동으로 발화하는 감사 훅이다. matcher 없이 등록되어 있어 어떤 종류의 서브에이전트가 시작되어도 예외 없이 발화한다.

이 훅의 목적은 SDI가 멀티에이전트 협업 흐름에서 어떤 서브에이전트가 어떤 태스크 컨텍스트 아래 기동되었는지 추적하는 것이다. SDI는 단일 에이전트 흐름을 안티패턴으로 간주하며(D13), 오케스트레이터가 전문화된 서브에이전트에게 작업을 위임하는 것이 기본 운영 모델이다. 이 훅은 그 위임의 시작 시점을 기록하여 이후 `SubagentStop` 훅과 짝을 이루어 서브에이전트 한 번의 실행 구간을 완전히 추적할 수 있게 한다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

matcher가 설정되지 않아 어떤 서브에이전트가 기동되어도 발화한다. 에이전트 이름이나 유형에 관계없이 적용된다.

**선결 조건**

환경 변수 `SDI_ACTIVE_TASK`가 설정되어 있지 않으면 훅은 아무 동작 없이 조용히 종료한다. 태스크 컨텍스트 없이 서브에이전트 기동을 기록하면 어떤 태스크에 연결해야 할지 알 수 없기 때문이다.

**기록 흐름**

1. `GET /projects/by-cwd`로 현재 작업 디렉토리에 연결된 프로젝트 ID를 조회한다.
2. `POST /activity`로 서브에이전트 기동 활동을 데몬에 전송한다. 전송하는 페이로드에는 프로젝트 ID, 활동 종류(`subagent.start`), 에이전트 이름(네임스페이스 접두사가 제거된 단순 이름), 그리고 `SDI_ACTIVE_TASK`에서 읽은 태스크 ID(`entity_id`)가 포함된다.

에이전트 이름에 포함된 네임스페이스 접두사(예: `sdi:impl-coder`에서 `sdi:` 부분)는 기록 전에 제거된다. 데몬에 저장되는 이름은 `impl-coder`와 같이 단순화된 형태다.

**출력 없음**

이 훅은 Claude Code의 세션 컨텍스트에 아무것도 주입하지 않는다. 서브에이전트 자신이나 오케스트레이터 모두 이 훅의 실행을 인지하지 못하며, 완전히 백그라운드에서 동작하는 레코드 전용 훅이다.

## 권한 / 제약

이 훅은 `POST /activity` 하나의 쓰기 요청만 수행한다. 서브에이전트의 실행 흐름에 영향을 주지 않으며, 기록 실패가 서브에이전트의 작업을 중단시키지 않는다.

`SDI_ACTIVE_TASK` 미설정 시 조용히 무시한다는 동작은 의도적 설계다. 오케스트레이터가 태스크를 설정하지 않은 상태에서 서브에이전트를 기동하는 흐름(비상 우회나 비정규 사용 등)에서 이 훅이 오류를 발생시켜 에이전트 기동 자체를 방해하는 일이 없도록 하기 위함이다.

`SDI_BYPASS_HOOKS=1` 환경 변수가 설정된 경우 이 훅을 포함한 모든 훅이 우회된다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: 없음(모든 서브에이전트 기동에 적용)
- 사용하는 데몬 엔드포인트: `GET /projects/by-cwd`, `POST /activity`
- 전송하는 활동 종류: `subagent.start`
- 선결 조건: `SDI_ACTIVE_TASK` 환경 변수 필수
- 짝이 되는 훅: `endpoint.hook-subagent-stop`
- 관련 설계 결정: D13 (멀티에이전트 오케스트레이션 우선), D15 (4가지 내장 멀티에이전트 패턴), D21 (위임 게이트), D22 (CollaborationPattern 엔티티)

## 미확정 (OPEN)

- `SDI_ACTIVE_TASK` 미설정 시 조용한 종료가 감사 로그에 남는지 완전 무시인지 미확인
- 네임스페이스 제거 로직의 구체적인 구분자 패턴 미확인 (`:` 기준인지, `/` 기준인지)
- 데몬 `POST /activity` 호출 실패 시 재시도 로직 존재 여부 미확인
- `SDI_ACTIVE_SCENARIO`와 같은 별도 환경 변수가 페이로드에 포함되는지 미확인
- 여러 서브에이전트가 동시에 기동될 때 기록 순서 보장 여부 미확인
