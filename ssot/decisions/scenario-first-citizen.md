---
id: decision.scenario-first-citizen
kind: Decision
title: GWT 시나리오를 제품의 1등 시민으로 둔다
purpose: "구현·검증·회귀 점검의 단위를 무엇으로 정할지 — Task/코드 기반으로 갈지, 자연어 GWT 시나리오로 갈지"
definition: "Plan이 잠그는 단위, LLM이 구현·검증하는 단위, Round가 재실행하는 단위는 모두 Given/When/Then 시나리오다. 코드·Task가 아니라 시나리오가 제품 전체의 1등 시민 개념이다."
relatesTo:
  - { to: concept.scenario, type: governs, note: "이 결정이 Scenario 개념의 지위를 1등 시민으로 격상한다" }
  - { to: concept.given-when-then, type: governs, note: "GWT 형식 자체가 검증 가능 단위의 형태적 요건이다" }
  - { to: domain.scenario-management, type: impacts, note: "시나리오를 1등 시민으로 두어 이 도메인 전체가 성립한다" }
  - { to: domain.round-execution, type: impacts, note: "Round가 재실행하는 단위가 시나리오이므로 회귀 검증 도메인의 기반" }
  - { to: domain.task-decomposition, type: impacts, note: "Task는 시나리오에서 LLM이 파생하는 런타임 산출물로 격하" }
  - { to: platform.sdi, type: impacts, note: "SDI의 정체성 명제 자체가 이 결정 위에 서 있다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI의 선행 도구인 Clawket v3.0은 Task 중심 모델이었다. Task의 "done" 기준이 자유 형식 evidence string이라 강제력은 있어도, evidence 내용 자체가 자유로워 회귀 검증을 자동화할 단위가 없었다. 이는 Jira/Linear/Notion과 같은 인간 중심 작업 관리 모델의 한계와 동일했다.

TDD는 검증 단위가 코드(test)라 자연어 의도와의 갭이 사람 머릿속에만 있고, BDD(Cucumber 등)는 Gherkin DSL + step definition 유지보수 비용이 크다. LLM 시대에는 LLM이 자연어 명세를 직접 읽고 검증 가이드로 쓸 수 있으므로, 컴파일 단계 없이 자연어 GWT가 검증 단위가 되는 것이 가능해졌다.

## 결정 (Decision)

GWT(Given/When/Then) 시나리오를 SDI의 1등 시민으로 정한다. 즉 다음이 모두 시나리오 단위로 이루어진다.

- Plan 승인 게이트: 유효한 GWT 시나리오가 1개 이상일 때만 Plan이 승인된다(Task 개수와 무관).
- LLM의 구현 단위: LLM specialist 에이전트가 시나리오를 읽고 구현하며, 시나리오가 명세이자 검증 기준이 된다.
- Round의 회귀 단위: R2 이상의 Round는 이전 모든 Round의 모든 시나리오를 재실행한다.
- Disruption 탐지 단위: 새 요구사항·결정이 기존 시나리오를 무효화할 때 disruption 리뷰가 트리거된다.

Task는 1등 시민이 아니라 LLM이 시나리오로부터 런타임에 파생하는 2차 산출물이다.

## 근거와 결과 (Consequences)

시나리오를 1등 시민으로 두면 다음 세 가지가 함께 성립한다.

첫째, 검증 가능한 단위가 자연어다. LLM이 GWT를 직접 읽어 구현·검증하므로 step definition 유지보수 없이 BDD의 의도를 실현한다.

둘째, 회귀 자동화의 단위가 생긴다. 이전 시나리오 전부를 다음 Round에서 재실행하는 strict-regression이 구조적으로 가능해진다. 자유 형식 evidence 시대에는 회귀 기준이 없어 자동 재실행이 불가능했다.

셋째, "결정이 무엇을 깨는가"를 구조적으로 추적할 수 있다. 시나리오가 DB에 영속화된 단위이므로, 새 요구사항·결정과의 영향 관계를 그래프로 조회할 수 있다.

대가로 받아들인 트레이드오프는, 모든 요구사항이 GWT 형식으로 정형화되어야 한다는 진입 비용이다. 자유 자연어 → GWT 변환을 LLM이 보조하도록 설계하여 이 비용을 낮췄다.

<!-- provenance: sdi-plugin/docs/PRD.md §1.2, §2 D1·D2·D5·D8 — GWT 1등 시민화, Plan approve 게이트, Task 런타임 산출물. sdi-plugin/CLAUDE.md D1·D2·D5·D8. sdi-plugin/README.md "Seven first-class entities" 섹션 -->
