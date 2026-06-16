---
id: endpoint.hook-session-start
kind: Endpoint
title: Hook `SessionStart` (세션 시작 컨텍스트 주입 및 데몬 기동)
definition: "Claude Code 세션이 열릴 때 SDI 데몬 상태를 점검하고, 활성 플랜·시나리오·태스크 요약을 컨텍스트로 주입하는 생명주기 훅"
realizedBy: []
implementedIn:
  - "sdi-plugin/plugin/hooks/hooks.json"
  - "sdi-plugin/plugin/adapters/claude/session-start.cjs"
  - "sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs"
relatesTo:
  - { to: "concept.plan", type: "reads" }
  - { to: "concept.scenario", type: "reads" }
  - { to: "concept.task", type: "reads" }
  - { to: "concept.decision", type: "reads" }
  - { to: "concept.project", type: "reads" }
  - { to: "domain.plugin-runtime", type: "belongs-to" }
  - { to: "domain.planning", type: "supports" }
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

`SessionStart` 훅은 Claude Code 세션이 새로 열리거나 `/clear`, `/compact` 명령으로 세션 컨텍스트가 초기화될 때 자동으로 발화하는 생명주기 훅이다. 이 훅의 핵심 역할은 두 가지다. 첫째, SDI 플러그인이 올바르게 동작하는 데 필요한 바이너리(`sdi`, `sdid`)와 스킬 파일이 모두 설치되어 있는지, 그리고 백그라운드 데몬이 살아 있는지를 확인한다. 둘째, 확인이 끝나면 현재 작업 디렉토리에 연결된 프로젝트의 활성 플랜 상태, 시나리오 현황, 진행 중인 태스크 목록, 최근 의사결정 수를 세션 컨텍스트로 주입하여 에이전트가 즉시 작업을 재개할 수 있도록 돕는다.

이 훅이 없으면 에이전트는 매 세션마다 맥락을 잃고 다시 파악하는 데 시간을 소비하거나, 데몬이 내려간 상태에서 작업을 시도하여 의도치 않은 오류가 발생할 수 있다. SDI의 태스크 주도 개발 원칙은 세션 경계를 넘어서도 연속성을 보장해야 하며, 이 훅이 그 연속성의 진입점 역할을 담당한다.

## 발화 조건 / 주입·차단 결과

**발화 조건**

훅 등록에 사용된 matcher 패턴은 `startup|clear|compact`로, Claude Code가 다음 세 가지 이벤트를 SessionStart로 보고할 때 이 훅이 실행된다. 첫 번째는 Claude Code 프로세스가 처음 시작될 때이고, 두 번째는 사용자가 `/clear`로 대화 히스토리를 비울 때이며, 세 번째는 `/compact`로 컨텍스트를 압축할 때다. 세 가지 모두 에이전트가 직전 상태를 잃는 전환점이므로 SDI 상태를 재주입해야 하는 시점이 된다.

**점검 순서**

훅 스크립트는 다음 순서로 환경을 점검한다.

1. 바이너리 설치 게이트: `sdi` 및 `sdid` 실행 파일과 4개의 SKILL.md 파일이 플러그인 경로에 존재하는지 확인한다. 설치가 불완전하면 사용자에게 설치 안내를 주입하고 이후 점검을 건너뛴다.
2. 데몬 헬스 체크: `GET /health` 엔드포인트로 데몬가 응답하는지 확인한다. 응답이 없으면 데몬을 자동으로 기동한 뒤 재확인한다.
3. 프로젝트 조회: `GET /projects/by-cwd`로 현재 작업 디렉토리에 연결된 프로젝트를 조회한다. 등록된 프로젝트가 없으면 등록 방법을 안내하는 메시지를 컨텍스트로 주입하고 이후 단계를 건너뛴다.
4. 핸드오프 데이터 수집: `GET /projects/:id/handoff`를 호출하여 활성 플랜의 단축 코드, 상태별 시나리오 수(confirmed/draft/retired), 진행 중인 태스크 최대 3건의 미리보기, 최근 의사결정 수, 데몬이 추천하는 다음 단계, 최근 활동 요약을 한꺼번에 가져온다. 추가로 `GET /projects/:id/next`로 에이전트에게 제안할 다음 행동을 조회한다.

**주입 결과**

수집된 정보는 `additionalContext` JSON 필드에 담겨 Claude Code의 세션 컨텍스트로 주입된다. 에이전트는 이 정보를 바탕으로 어떤 플랜이 활성화되어 있는지, 어떤 태스크가 진행 중인지, 그리고 데몬이 제안하는 다음 행동이 무엇인지 즉시 파악할 수 있다.

프로젝트가 미등록 상태라면 플랜 정보 대신 프로젝트 등록 안내가 주입된다. 바이너리가 누락된 경우에는 설치 지침이 먼저 표시된다.

**감사 로그**

발화 시마다 `~/.local/state/sdi/hook.log`에 `session_start` 이벤트 유형으로 감사 항목이 기록된다. 이를 통해 훅이 언제, 얼마나 자주 발화했는지 추적할 수 있다.

## 권한 / 제약

이 훅은 읽기 전용이다. 데이터를 수정하거나 태스크를 생성하는 부작용이 없으며, 단 하나의 부작용인 데몬 자동 기동은 기존에 데몬이 실행되지 않은 경우에만 발생한다.

데몬이 이미 실행 중이면 `GET /health` 응답만으로 통과하며 별도의 재기동을 시도하지 않는다. 데몬 기동에 실패하면 이후 상태 주입 단계를 건너뛰고 오류 맥락을 주입하여 사용자가 인지할 수 있도록 한다.

이 훅은 Claude Code의 세션 경계마다 실행되므로 지연이 누적될 수 있다. 데몬 헬스 체크와 API 호출의 타임아웃 정책이 중요하지만, 현재 코드 확인 전까지는 구체 값이 미확정이다.

## provenance

- 출처: `sdi-plugin/plugin/hooks/hooks.json` 및 `sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs` 분석에서 추론
- matcher 패턴: `startup|clear|compact`
- 사용하는 데몬 엔드포인트: `GET /health`, `GET /projects/by-cwd`, `GET /projects/:id/handoff`, `GET /projects/:id/next`
- 감사 로그 경로: `~/.local/state/sdi/hook.log`
- 관련 설계 결정: D1 (도구 정체성), D3 (태스크는 런타임 아티팩트), D13 (멀티에이전트 오케스트레이션 우선)

## 미확정 (OPEN)

- `GET /projects/:id/handoff`와 `GET /projects/:id/next`가 단일 호출인지 두 번의 별도 호출인지 코드 확인 필요
- 데몬 자동 기동 방식(exec 방식, 재시도 횟수, 대기 시간)이 구체적으로 어떻게 구현되어 있는지 미확인
- 주입되는 in-flight 태스크 미리보기의 최대 건수가 3건으로 고정인지 설정 가능한지 미확인
- 바이너리 4개의 SKILL.md가 구체적으로 어떤 파일인지 목록 미확인
- 데몬 헬스 체크 및 데이터 수집 각 단계의 타임아웃 임계값 미확인
