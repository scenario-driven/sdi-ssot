---
id: decision.gwt-strict
kind: Decision
title: GWT 형식을 엄격하게 강제하고 자유 형식 시나리오를 허용하지 않는다
purpose: "시나리오 입력 형식을 얼마나 강제할지 — 자유 형식을 허용할지, Given/When/Then을 필수로 강제할지"
definition: "모든 시나리오는 Given·When·Then 세 필드가 비어 있으면 저장을 거부한다. 자유 형식 시나리오 옵션은 없다. 자유 자연어 → GWT 변환은 LLM이 보조한다."
relatesTo:
  - { to: concept.given-when-then, type: governs, note: "이 결정이 GWT 개념의 엄격성 요건을 정의한다" }
  - { to: concept.scenario, type: governs, note: "모든 Scenario 인스턴스는 이 결정의 형식 강제를 따른다" }
  - { to: domain.scenario-management, type: impacts, note: "시나리오 저장·검증 경로 전체가 이 결정 위에서 동작한다" }
  - { to: domain.round-execution, type: impacts, note: "회귀 재실행의 단위가 GWT 완비 시나리오이므로 Round 실행에 직접 영향" }
  - { to: domain.planning, type: impacts, note: "Plan approve 게이트는 GWT 유효 시나리오 존재를 전제한다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

시나리오가 검증 단위의 1등 시민이 되려면 시나리오 자체가 기계적으로 판별 가능한 형식을 가져야 한다. BDD의 Gherkin은 이를 DSL로 풀었지만 step definition 유지보수 비용이 컸다. SDI에서는 LLM이 자연어 GWT를 직접 읽어 구현·검증 가이드로 쓰므로, 자연어이면서도 Given/When/Then 세 섹션이 명확히 분리되어 있어야 LLM이 각 부분의 역할을 혼동 없이 처리할 수 있다.

자유 형식 옵션을 허용하면, 개발자 편의는 높아지지만 "형식이 맞는가"를 LLM이 매번 추론해야 하고, 회귀 재실행 시 "이 시나리오가 충족됐는가"를 판단할 공통 기준이 없어진다.

## 결정 (Decision)

시나리오 CRUD 검증 계층에서 Given / When / Then 세 필드가 하나라도 비어 있으면 저장을 거부한다. 자유 형식(free-form) 시나리오 옵션은 도구 어디에도 존재하지 않는다.

자유 자연어로 동작을 설명하는 개발자를 위해, LLM이 그 설명을 GWT로 변환하는 보조 흐름을 `/scenario` 슬래시 명령에 제공한다. 변환 결과를 사람이 확인한 뒤 저장하는 방식으로 편의와 엄격성을 동시에 충족한다.

## 근거와 결과 (Consequences)

GWT 형식이 완비되어야만 다음이 성립한다.

- Plan approve 게이트: GWT 유효 시나리오가 1개 이상인지를 검사 가능해진다.
- Round 회귀: R2+가 "모든 이전 시나리오"를 재실행할 때, 각 시나리오의 Given(사전 조건)·When(실행 조건)·Then(기대 결과)을 LLM이 명확히 구분해 실행할 수 있다.
- Disruption 탐지: 새 변경이 기존 시나리오의 Then(기대 결과)을 무효화하는지를 구조적으로 판단할 수 있다.

형식 강제의 부담은 LLM 보조 변환으로 완화된다. 자유 형식 시나리오가 쌓여 "검증 단위로 쓸 수 없는 시나리오"가 DB에 혼재하는 상황을 설계 단계에서 차단하는 것이 이 결정의 핵심 가치다.

<!-- provenance: sdi-plugin/docs/PRD.md §2 D5 "GWT 형식 강제. 자유 형식 옵션 없음". sdi-plugin/CLAUDE.md D5. crates/core/ scenario CRUD validation (GWT 비어있으면 거부). -->
