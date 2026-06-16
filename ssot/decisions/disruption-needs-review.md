---
id: decision.disruption-needs-review
kind: Decision
title: 새 시나리오·요구사항·결정이 기존 시나리오를 건드릴 때 기본 정책은 사람 확인이다
purpose: "기존 시나리오를 무효화·수정할 수 있는 변경이 들어올 때 어떻게 처리할지 — LLM이 자율로 적용할지, 사람 확인 후 진행할지"
definition: "Disruption(새 변경이 기존 시나리오를 무효화·수정해야 하는 상황)이 탐지되면 기본 정책은 needs-review로, LLM이 임의 적용하지 않고 사람 확인을 요청한다. auto 옵션은 명시적 옵트인으로만 활성화된다."
relatesTo:
  - { to: concept.scenario, type: impacts, note: "기존 Scenario의 유효성이 disruption 검토의 대상이다" }
  - { to: domain.disruption-review, type: governs, note: "이 결정이 disruption 검토 도메인의 기본 동작을 정의한다" }
  - { to: domain.scenario-management, type: impacts, note: "시나리오 변경 경로에 disruption 게이트가 삽입된다" }
  - { to: domain.autonomy, type: impacts, note: "자율 적용 여부는 이 정책이 자율성 스펙트럼과 교차하는 지점이다" }
  - { to: domain.governance-audit, type: impacts, note: "disruption 이벤트와 사람 확인 기록이 감사 대상이다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI는 GWT 시나리오를 영속 단위로 관리하므로, 새 요구사항·결정·시나리오가 들어올 때 기존 시나리오와 충돌하거나 기존 시나리오를 무효화하는 상황이 발생한다. 이를 Disruption이라 부른다.

Disruption이 발생했을 때 LLM이 자율적으로 기존 시나리오를 수정·삭제하면 사람이 알아채지 못한 사이에 합의된 동작 규약이 바뀔 위험이 있다. 반대로 모든 disruption을 사람이 일일이 확인하면 LLM 자율 실행의 가치가 희석된다.

## 결정 (Decision)

Disruption 정책의 기본값은 needs-review(사람 확인 필수)로 정한다. 새 시나리오·요구사항·결정이 기존 시나리오의 GWT 의미와 충돌하거나 무효화할 가능성이 탐지되면, LLM은 자동 적용을 보류하고 사람 확인을 요청하는 게이트를 띄운다.

auto 모드(LLM이 스스로 disruption을 처리하고 변경 적용)는 명시적 옵트인으로만 활성화된다. auto 모드에서도 LLM이 결정을 내렸음을 기록하고, 이후 사용자가 확인할 수 있는 감사 이력을 남긴다.

## 근거와 결과 (Consequences)

needs-review를 기본으로 두는 이유는 두 가지다. 첫째, 합의된 동작 규약(시나리오)을 사람 확인 없이 변경하는 것은 SDI의 "시나리오 = 검증된 계약"이라는 정체성과 충돌한다. 둘째, AutonomyPolicy의 자율성 레벨(L3~L5)과 연계하여 disruption 처리 수준을 정책으로 조정할 수 있어야 한다.

대가로 disruption이 잦은 환경에서는 사람이 확인해야 할 게이트가 늘어나는 마찰이 생긴다. 이는 AutonomyPolicy를 통해 per-scope로 auto를 활성화하거나, disruption 판단 기준을 조정하여 완화한다. SDI가 "사람이 PC 앞에 묶이지 않는 것"을 목표로 하더라도, 기존 계약을 깨는 결정만큼은 사람이 최소한 한 번 인지하는 것이 옳다는 판단이다.

<!-- provenance: sdi-plugin/docs/PRD.md §2 D9 "Disruption 정책 기본: needs-review(사람 확인). 옵션: auto". sdi-plugin/CLAUDE.md D9. -->
