---
id: capability.convert-gwt
kind: Capability
title: 자연어 → Given/When/Then 변환
purpose: "사람이 자유롭게 말한 동작 설명을 LLM이 G/W/T 원자 형식으로 정규화해 시나리오 저장 전에 품질을 보증한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-scenario/SKILL.md
relatesTo:
  - { to: concept.given-when-then, type: mutates, note: "자연어를 받아 G/W/T 세 절로 분해·정제한다" }
  - { to: concept.scenario, type: mutates, note: "정규화 결과가 곧 등록할 시나리오의 내용이 된다" }
  - { to: concept.scenario-claim, type: relates-to, note: "한 시나리오 = 하나의 검증 가능한 단언이라는 원자성 규칙을 적용해 분리 판단한다" }
impacts:
  - concept.scenario
  - concept.given-when-then
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

사람이 "로그인 폼 빈칸 검사 넣어줘", "결제 실패 시 재시도 화면"처럼 자유로운 말로 기대 동작을 표현하면, LLM이 그것을 세 개의 명확한 절로 재구성한다. Given은 그 동작이 일어나기 위해 필요한 전제 상태, When은 단 하나의 트리거 행동, Then은 관찰 가능한 결과다. 사람은 GWT 문법을 몰라도 되고, LLM이 형식화 책임을 진다.

"검색이 빠르면 좋겠어"처럼 아직 측정 기준이 없는 막연한 표현은 시나리오가 아니다. 이 경우 LLM은 구체화를 요청해 "p95 응답 200ms 이하"처럼 관찰 가능한 형태로 만들고 나서 G/W/T로 변환한다.

## 행위

- 사용자 설명에서 행위자, 전제 상태, 트리거, 기대 결과를 추출한다.
- 복합 행위("그리고…", 여러 Then)는 독립적인 원자 시나리오로 분리한다.
- 각 절이 관찰 가능한 단언인지 검사한다—"잘 동작한다", "빠르다" 같은 비관찰 Then은 구체화 전에 등록하지 않는다.
- 정규화 결과를 `sdi scenario create`로 전달해 데몬 검증(`GWT_EMPTY` 거부)을 통과시킨다.

## 시스템 흐름

사용자 자연어 입력 → `sdi-scenario` 스킬의 LLM이 절 추출·정규화 → 원자성 검사로 분리 여부 결정 → 분리 시 여러 G/W/T 트리플 생성 → 각각 `sdi scenario create` 호출 → 데몬이 비어있지 않음을 최종 검증. 이 스킬은 저장 이전의 형식화 계층이다.

## 어디에 구현되어 있나

변환 로직은 `sdi-scenario` 스킬(`plugin/skills/sdi-scenario/SKILL.md`)에 정의된 LLM 측 정규화 지침이다. 데몬은 변환 로직 자체를 갖지 않고, 결과물의 구조(비어있지 않은 세 절)만 검증한다. 변환 품질의 책임은 LLM에 있다.

## 미확정 (OPEN)

- [ ] OPEN: LLM이 변환 중 사용자 확인이 필요하다고 판단할 때의 명시적 프롬프트 패턴 문서화 여부 확인 필요.
