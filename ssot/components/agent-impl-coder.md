---
id: component.agent-impl-coder
kind: SystemComponent
title: impl-coder 서브에이전트
purpose: 활성 태스크에 연결된 시나리오를 명세로 삼아 그 시나리오를 만족하는 최소한의 코드 변경을 구현하는 전문 에이전트다. 시나리오가 곧 스펙이고, 그 외 범위를 넘는 리팩토링이나 주변 코드 정리는 그 자체가 버그의 근본 원인일 때만 허용한다.
realizedBy:
  - domain.scenario-management
  - domain.task-decomposition
  - domain.round-execution
implementedIn:
  - sdi-plugin/plugin/agents/impl-coder.md
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
    note: impl-coder가 구현을 완료하면 test-runner에게 검증을 위임한다
  - to: component.agent-disruption-analyst
    type: leads-to
    note: 변경 범위가 링크된 시나리오 밖으로 퍼질 때 disruption-analyst에게 위임한다
  - to: domain.scenario-management
    type: backed-by
  - to: domain.task-decomposition
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

impl-coder는 SDI 구현 루프에서 "코드를 실제로 작성하는" 역할을 맡는 전문 에이전트다. 이 에이전트가 활성화되는 시점은 태스크가 `in_progress` 상태이고 해당 태스크에 시나리오가 연결되어 있을 때다.

작업 방식은 다음 순서를 강제한다. 먼저 태스크에 연결된 시나리오와 요구사항을 모두 읽는다 — 이것이 구현 명세다. 그다음 수정할 파일을 실제로 읽은 후에 편집한다(컨텍스트 만료를 막기 위한 fresh read). 구현이 끝나면 타입 체크(`cargo check --workspace` 등)를 돌려 컴파일이 통과하는 것을 확인한다. 이후 test-runner 에이전트에게 검증을 넘긴다.

시나리오의 자연 폭발 반경 밖의 파일은 건드리지 않는다. 단, 근본 원인이 구조적 결함인 경우 주변 정리가 곧 실제 수정이므로 그 경우에만 확장된 범위가 정당화된다.

## 경계와 의존

impl-coder는 `sdi` CLI를 통해 태스크·시나리오 정보를 읽고, 구현 후 `sdi task update` 등으로 진행 상태를 기록한다. Edit·Write 도구로 파일을 수정하며, 이 도구들은 D21 위임 게이트를 통과한 서브에이전트로서 실행 권한을 갖는다. 데몬에 직접 HTTP를 보내지 않고 CLI를 인터페이스로 사용한다.

## 통신 패턴

메인 세션 오케스트레이터가 impl-coder를 서브에이전트로 스폰하고, impl-coder는 작업 완료 후 결과를 메인 세션에 반환하거나 test-runner로 바턴을 넘긴다(직접 핸드오프 또는 agent-note 경유).

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/impl-coder.md`)로 구성된다. 별도 코드 모듈이 없으며, 에이전트 행동 규범·워크플로우·불변식이 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: impl-coder와 test-runner 사이의 핸드오프 채널이 agent-note인지 직접 메시지인지 구체 구현 확인 필요.
