---
id: component.agent-disruption-analyst
kind: SystemComponent
title: disruption-analyst 서브에이전트
purpose: 코드 변경이 활성 태스크에 연결된 시나리오 바깥에 영향을 미치는지 감지하고 분류하는 전문 에이전트다. 의도적 행동 변경은 decision row를 통해 정당화하고, 의도하지 않은 실패는 회귀(failing)로 표시해 test-runner에 넘긴다. D9 needs-review 정책의 집행자다.
realizedBy:
  - domain.disruption-review
  - domain.scenario-management
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/disruption-analyst.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.coding-agent
relatesTo:
  - to: component.agent-test-runner
    type: leads-to
    note: 회귀로 판정된 시나리오는 test-runner에게 failing 판정 기록을 위임한다
  - to: component.agent-decision-resolver
    type: leads-to
    note: 의도적 행동 변경은 decision-resolver를 통해 decision row를 만든다
  - to: domain.disruption-review
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

disruption-analyst는 "변경의 경계를 감시하는" 에이전트다. impl-coder나 test-runner가 변경이 링크된 시나리오 밖 시나리오를 건드렸다고 의심할 때 이 에이전트가 개입한다.

분석 절차는 다음과 같다. 활성 태스크에 연결된 시나리오 집합을 먼저 파악한다. 그다음 변경된 파일에서 다른 시나리오의 단코드, GWT 절, 테스트 이름 등의 참조를 검색한다. 각 후보 시나리오에 대해 의도적 변경인지 의도치 않은 파급인지 판별한다.

의도적으로 행동을 바꾼 경우에는 decision-resolver를 통해 decision row를 만들고, 그 decision id를 근거로 시나리오를 `impacted`로 표시한다. decision id 없는 `impacted` 표시는 인정되지 않는다. 의도치 않은 파급(회귀)이면 `failing`으로 표시하고 test-runner에 넘긴다.

D9 정책(needs-review 기본값)에 따라 자동 처리(`auto` 옵션)가 설정되어 있어도 사람 확인이 필수다. 에이전트가 혼자 판정을 적용하고 넘어가는 것은 프로토콜 위반이다.

## 경계와 의존

Bash·Read 도구로 코드와 SDI 상태를 읽는다. 코드 파일을 수정하지 않는다. CLI를 통해 시나리오 상태를 조회하고 decision 생성을 decision-resolver에 위임한다.

## 통신 패턴

impl-coder 또는 test-runner가 혼란을 감지했을 때 에스컬레이션 경로로 활성화된다. 분석 완료 후 판정과 근거를 메인 세션에 반환하거나 decision-resolver·test-runner로 각 케이스를 위임한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/disruption-analyst.md`)로 구성된다. 감지 워크플로우·판정 기준·불변식이 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: "링크된 시나리오 바깥 시나리오" 감지를 위한 grep 방법론 — short code 기준인지, GWT 텍스트 패턴 매칭인지, 테스트 파일 이름 매칭인지 — 이 에이전트 명세 이상으로 구체화될 필요가 있다.
