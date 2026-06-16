---
id: decision.strict-regression-default
kind: Decision
title: Round 기본 모드는 strict-regression으로 이전 모든 시나리오를 자동 재실행한다
purpose: "R2 이상의 Round를 시작할 때 이전 Round의 시나리오를 어떻게 처리할지 — 새 시나리오만 실행할지, 이전 시나리오를 모두 재실행할지"
definition: "Round의 기본 모드는 strict-regression이다. R2 이상에서 Round를 시작하면 이전 모든 Round의 모든 시나리오를 자동 재실행하여 회귀 여부를 확인한다. forward-only는 명시적 옵트인으로만 선택 가능하다."
relatesTo:
  - { to: concept.round, type: governs, note: "Round 기본 동작 방식을 이 결정이 정의한다" }
  - { to: concept.regression, type: governs, note: "자동 회귀 검증 정책의 근거다" }
  - { to: domain.round-execution, type: impacts, note: "Round 시작 API와 실행 엔진의 기본 동작을 규정한다" }
  - { to: domain.scenario-management, type: impacts, note: "모든 Scenario가 이후 Round에서 회귀 대상으로 보존된다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket v3.0에서 회귀 검증은 사람이 수동으로 트리거해야 했다. LLM의 sub-agent 병렬 실행 능력을 활용해 회귀를 자동화하는 것이 SDI 설계의 핵심 동기 중 하나였다.

회귀 정책의 선택지는 두 가지다. 하나는 새 Round에서 새 시나리오만 실행하고 이전 시나리오는 건드리지 않는 forward-only 방식이다. 다른 하나는 이전 모든 시나리오를 다시 실행해 "아무것도 깨지지 않았음"을 확인하는 strict-regression 방식이다. 사용자가 잊었거나 알아차리지 못한 회귀를 도구가 자동으로 잡아줘야 한다는 요구가 있었다.

## 결정 (Decision)

Round의 기본 모드를 strict-regression으로 정한다. R2 이상에서 `sdi round start`를 실행하면, 이전 모든 Round에서 통과한 모든 시나리오가 자동으로 재실행 대상에 포함된다. 새 시나리오와 이전 시나리오를 별도 엔진이 처리하는 것이 아니라 같은 엔진이 처리한다(R1 = 신규 개발 모드, R2+ = 회귀 포함 실행 모드이나 엔진 코드는 동일).

이전 시나리오를 재실행하지 않으려면 `--mode forward-only` 플래그를 명시적으로 지정해야 한다. 기본값으로 forward-only를 선택하는 것은 사용자가 의식하지 못하는 회귀를 놓치는 위험을 낳으므로 금지한다.

## 근거와 결과 (Consequences)

strict-regression을 기본으로 두면 다음이 따라온다.

- LLM 에이전트는 새 작업이 이전에 합의된 동작을 깨지 않았음을 Round 시작 시 자동으로 검증한다. 개발자가 회귀 검증을 별도로 지시하지 않아도 된다.
- 회귀가 발생하면 해당 시나리오가 disruption 검토 대상이 되어 사람 확인 게이트(D9 needs-review)가 트리거된다.
- Round 실행 시간이 시나리오 누적에 따라 늘어날 수 있다. 이는 sub-agent 병렬 실행으로 완화하며, "속도보다 안전성"을 우선하는 의도적 트레이드오프다.
- forward-only는 탈출 옵션이지 권장 경로가 아님을 명시하고, 대시보드에서 모드를 시각적으로 구분한다.

<!-- provenance: sdi-plugin/docs/PRD.md §2 D6·D7 "Round 모드 기본값: strict-regression", "두 모드는 같은 엔진". sdi-plugin/CLAUDE.md D6·D7. -->
