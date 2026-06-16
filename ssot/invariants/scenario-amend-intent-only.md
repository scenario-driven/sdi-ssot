---
id: invariant.scenario-amend-intent-only
kind: Invariant
title: 시나리오 수정은 의도(intent) 변경일 때만 허용된다
definition: "Scenario의 GWT 내용 수정은 '의도가 바뀌었다'는 명시적 근거가 있을 때만 허용된다. 구현 방법의 변경이나 기술적 접근 변경을 이유로 시나리오 자체를 수정하는 것은 허용되지 않으며, 그 경우 시나리오는 그대로 두고 Decision과 새 Task를 통해 구현을 조정한다."
governs:
  - concept.scenario
  - concept.given-when-then
  - concept.scenario-claim
  - domain.scenario-management
  - domain.disruption-review
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI에서 시나리오는 "무엇을 해야 하는가"에 대한 자연어 계약이다. 시나리오의 GWT 내용을 바꾸는 것은 그 계약이 변경됐다는 의미다. 계약이 변경되는 상황은 오직 하나다: 사람 또는 도메인의 **의도(intent)** 자체가 바뀐 것. 구현 방식, 기술 선택, 내부 설계를 바꾸는 것은 시나리오를 수정하는 이유가 되지 않는다.

시나리오를 덮어쓰는 것은 회귀 검증의 역사적 기준점을 파괴한다. R2+ 라운드는 이전 라운드의 passing 시나리오를 재검증하는데, 시나리오 내용이 구현 편의를 위해 조용히 바뀌었다면 "무엇에 대한 회귀를 검증하는가"가 불명확해진다.

새 기능 또는 변경된 의도가 기존 시나리오에 영향을 미치는 경우 Disruption Review(`concept.disruption-review`) 흐름이 작동하며, 사람이 기존 시나리오를 retire/edit/keep 중 선택한다. 이것이 의도 변경을 처리하는 유일한 공식 경로다.

## 깨지면 무슨 일이 일어나나

구현 변경을 이유로 시나리오를 수정하면 R2+ 회귀가 의미를 잃는다. 이전 라운드에서 passing이었던 시나리오가 지금도 passing으로 나오는 것이 "구현이 올바르다"는 증거가 아니라 "시나리오를 구현에 맞게 바꿨다"는 신호가 될 수 있다. Disruption 정책(`invariant.strict-regression-default`)이 있음에도 조용히 시나리오를 수정하면 영향 시나리오가 needs-review 게이트를 거치지 않고 변경된다. 또한 `per_round_results`의 이전 passing 기록이 바뀐 시나리오 내용과 의미적으로 불일치하게 된다.

## 코드에서 어떻게 강제되나

데몬의 시나리오 수정 경로(`crates/daemon/src/router/scenario.rs`)는 Disruption 정책(D9)에 따라 기존 시나리오를 변경하면 영향 받는 라운드의 `disruption_review` 행을 생성한다. `has_pending` 검사가 `round activate`를 차단하여(`DISRUPTION_PENDING` 오류) 사람의 명시 결정 없이 영향이 조용히 진행되지 않는다. 자동화 경로(`auto` 모드)에서도 시나리오를 직접 폐기·수정하기 전에 사람 confirm을 요구하는 것이 PRD D9에 명시되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D9 Disruption needs-review 기본값 결정 엔트리) 연결 필요
