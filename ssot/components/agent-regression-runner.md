---
id: component.agent-regression-runner
kind: SystemComponent
title: regression-runner 서브에이전트
purpose: R2 이상 라운드에서 이전 라운드의 passing 시나리오를 현재 코드로 재실행해 회귀가 없는지 확인하는 전문 에이전트다. D6 strict-regression 기본값의 집행자이며, R1(신규 개발) 컨텍스트에서는 존재하지 않는다.
realizedBy:
  - domain.round-execution
  - domain.scenario-management
implementedIn:
  - sdi-plugin/plugin/agents/regression-runner.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-disruption-analyst
    type: leads-to
    note: passing이었던 시나리오가 failing이 되면 disruption-analyst에게 넘긴다
  - to: domain.round-execution
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

regression-runner는 R2+ 라운드에서만 동작하는 에이전트다(D7). R1은 신규 개발, R2 이상은 회귀 검증이라는 SDI의 라운드 의미론에서 이 에이전트의 역할이 정해진다.

동작 방식은 다음과 같다. 이전 완료된 라운드에서 `passing` 판정을 받은 시나리오 목록을 가져온다. 각 시나리오에 대해 현재 코드베이스로 관련 테스트를 실행한다. 이전에 통과했던 시나리오가 지금도 통과하면 carry를 확정하고, 실패하면 `failing`으로 표시한다.

엄격한 구분이 있다. `failing`은 "의도치 않은 회귀"이고, `impacted`는 "의도적으로 행동을 바꿨으며 decision row가 뒷받침한다"는 다른 판정이다. 이전에 통과했던 시나리오가 실패했는데 decision row가 없다면 절대 `impacted`로 표시하면 안 된다 — `failing`이다.

전체 이전 라운드 시나리오 집합을 실행해야 한다. 임의로 일부를 건너뛰는 것은 프로토콜 위반이다. 단, 시나리오 폐기가 필요하다면 disruption-analyst를 통해 진행한다.

## 경계와 의존

Bash·Read 도구만 갖는다. 코드를 수정하지 않으며 테스트를 실행하고 관찰한다. `sdi round results <PRIOR-ROUND-ID>`로 이전 라운드 판정을 조회하고, `sdi round result`로 현재 라운드에 증거를 기록한다.

## 통신 패턴

라운드 활성화(`sdi round activate`) 직후 오케스트레이터가 이 에이전트를 스폰한다. 실행 완료 후 전체 판정 요약을 반환하고, failing 시나리오가 있으면 disruption-analyst로 위임하거나 사용자에게 보고한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/regression-runner.md`)로 구성된다. R2+ 전용 불변식·판정 기준·워크플로우가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: `strict-regression` 모드와 `forward-only` 모드(D6 선택적 옵션)에서 regression-runner의 행동 차이가 있는지, forward-only에서는 이 에이전트를 스폰하지 않는지 확인 필요.
