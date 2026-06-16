---
id: component.agent-scenario-decomposer
kind: SystemComponent
title: scenario-decomposer 서브에이전트
purpose: 승인된 시나리오를 실행 가능한 태스크로 분해하는 전문 에이전트다. 태스크는 LLM이 시나리오와 요구사항에서 도출하는 런타임 아티팩트(D3)이며 사람이 직접 작성하지 않는다. 각 태스크는 최소 하나의 시나리오에 반드시 연결되어야 한다.
realizedBy:
  - domain.task-decomposition
  - domain.scenario-management
  - domain.round-execution
implementedIn:
  - sdi-plugin/plugin/agents/scenario-decomposer.md
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
    type: leads-to
    note: 분해된 태스크는 impl-coder가 구현한다
  - to: domain.task-decomposition
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

scenario-decomposer는 SDI 실행 루프의 시작점을 준비하는 에이전트다. 플랜에 있는 `confirmed` 상태의 시나리오만을 대상으로 태스크를 제안하고 생성한다. 미확정(unconfirmed) 시나리오는 사용자가 아직 동작 명세를 승인하지 않은 것이므로 분해 대상이 아니다.

각 시나리오마다 1~3개의 후보 태스크 제목을 제안한다. 하나의 라운드로 증거를 만들어낼 수 있을 만큼 작아야 한다. 시나리오가 너무 크면 `sdi task decompose`로 하위 태스크로 나눈다.

태스크 생성 시 다음을 반드시 충족해야 한다. 태스크는 최소 하나의 시나리오와 연결되어야 한다(`--scenario SCN-…`). 현재 활성 라운드 ID를 `sdi round active <PLAN-ID>`로 가져와 태스크에 귀속시킨다. 요구사항 연결(`--req REQ-…`)은 선택이지만 권장한다.

## 경계와 의존

Bash·Read 도구만 갖는다. 코드를 수정하지 않으며, `sdi scenario list`, `sdi round active`, `sdi task create` CLI 명령으로 시나리오를 읽고 태스크를 생성한다.

## 통신 패턴

시나리오 승인(confirmed) 후 아직 태스크가 없을 때 오케스트레이터가 이 에이전트를 스폰한다. 태스크 생성 완료 후 생성된 태스크 ID 목록과 각 시나리오 연결 관계를 반환한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/scenario-decomposer.md`)로 구성된다. 분해 불변식·선택 기준·워크플로우가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: `sdi task decompose` 서브커맨드의 실제 구현 여부와 분해 깊이 제한 확인 필요.
