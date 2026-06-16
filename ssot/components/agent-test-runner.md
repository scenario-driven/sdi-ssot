---
id: component.agent-test-runner
kind: SystemComponent
title: test-runner 서브에이전트
purpose: impl-coder가 구현을 완료한 뒤 프로젝트 테스트 스위트를 실행하고, 태스크에 연결된 모든 시나리오에 대해 정형화된 증거 행(evidence row)을 생성하는 전문 에이전트다. 판정 어휘는 passing·failing·impacted·retired 네 가지로 고정되며 skipped는 없다.
realizedBy:
  - domain.round-execution
  - domain.scenario-management
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/test-runner.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-impl-coder
    type: preceded-by
    note: impl-coder가 코드를 작성한 뒤 test-runner가 검증한다
  - to: component.agent-disruption-analyst
    type: leads-to
    note: 예상 밖 시나리오가 실패하면 disruption-analyst에게 넘긴다
  - to: domain.round-execution
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

test-runner는 SDI 검증 루프의 "결과를 측정하고 기록하는" 역할을 맡는다. 이 에이전트는 impl-coder가 코드 변경을 완료한 후에 활성화된다.

핵심 산출물은 **증거 행(evidence row)**이다. 태스크에 연결된 시나리오마다 반드시 하나의 판정을 내려야 한다. 판정 어휘는 네 가지다. `passing`은 테스트가 통과한 시나리오, `failing`은 테스트가 실패한 시나리오(회귀 포함), `impacted`는 이번 변경이 의도적으로 행동을 바꾼 시나리오(반드시 decision row가 뒷받침해야 한다), `retired`는 새 흐름으로 대체되어 폐기된 시나리오다. `skipped`는 존재하지 않으며, 부분 증거는 태스크 완료 시 데몬이 거부한다.

증거 참조(evidence reference)는 반드시 파일:라인 형식이거나 CI URL이어야 한다. 산문 설명만으로는 증거가 되지 않는다.

## 경계와 의존

test-runner는 read 전용 도구(Bash, Read)만 갖는다. 코드를 수정하지 않으며, 테스트를 실행하고 결과를 관찰하는 것이 전부다. `sdi round result` CLI 명령으로 데몬에 증거 행을 기록한다.

## 통신 패턴

impl-coder로부터 바턴을 받아 활성화된다. 판정 완료 후 결과를 메인 세션에 반환하거나, `failing`이나 예상 밖 `impacted`가 발생하면 disruption-analyst에게 위임한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/test-runner.md`)로 구성된다. 판정 어휘·증거 형식·워크플로우·불변식이 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: test-runner가 CI 환경에서 자동화 테스트를 직접 트리거하는 경로가 있는지, 아니면 로컬 테스트 실행만 지원하는지 확인 필요.
