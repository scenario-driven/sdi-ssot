---
id: component.agent-gwt-converter
kind: SystemComponent
title: gwt-converter 서브에이전트
purpose: 자유형 행동 설명 텍스트를 D5 규칙을 만족하는 Given/When/Then 시나리오로 변환하는 전문 에이전트다. 사용자가 예상 동작을 스케치하면 이 에이전트가 세 절 모두 비어 있지 않은 형식으로 정제하고 데몬에 기록한다.
realizedBy:
  - domain.scenario-management
implementedIn:
  - sdi-plugin/plugin/agents/gwt-converter.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.scenario-management
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

gwt-converter는 사람이나 에이전트가 자유로운 언어로 서술한 기대 동작을 SDI 시나리오 형식으로 바꾸는 에이전트다. 사용 시점은 SDI 플랜에 새 시나리오 행을 추가해야 하는데, 작성자가 아직 GWT 형식으로 정제하지 못한 상태일 때다.

변환 규칙은 엄격하다. Given(전제 조건), When(트리거), Then(관찰 가능한 결과) 세 절이 모두 비어 있으면 안 되며(D5), 이는 데몬이 시나리오 생성 시 강제 검증한다. 자연어 명세가 곧 스펙이므로 Gherkin step definition이나 테스트 프레임워크 DSL로 변환하지 않는다. 사용자가 쓴 도메인 어휘를 그대로 유지하며, 새 명사를 임의로 만들지 않는다.

한 설명에 두 개의 행동이 섞여 있으면, 시나리오를 하나로 뭉치지 않고 두 개로 분리할 것을 제안한다. 한 시나리오 = 하나의 관찰 가능한 행동이 원칙이다.

절 중 어느 하나라도 불명확하면 추론하지 않고 작성자에게 하나의 집중 질문을 한다.

## 경계와 의존

Bash·Read 도구만 갖는다. 코드를 수정하지 않으며, `sdi scenario create` CLI 명령으로 데몬에 시나리오를 기록한다.

## 통신 패턴

사용자 또는 오케스트레이터가 자유 텍스트 설명과 함께 이 에이전트를 스폰한다. GWT 트리플이 완성되면 CLI로 시나리오를 생성하고 생성된 시나리오 ID를 반환한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/gwt-converter.md`)로 구성된다. 변환 원칙·불변식·워크플로우가 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: GWT 절이 너무 길거나 복잡한 경우 분할 제안의 판단 기준이 에이전트 자율 판단인지 명세가 있는지 확인 필요.
