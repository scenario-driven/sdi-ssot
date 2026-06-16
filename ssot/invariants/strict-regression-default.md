---
id: invariant.strict-regression-default
kind: Invariant
title: R2+ 라운드의 기본 회귀 모드는 strict-regression이다
definition: "R2 이상의 Round를 시작할 때 기본 Round 모드는 strict-regression이다. 이 모드에서는 이전 모든 라운드(R1..R(N-1))의 passing 시나리오 전부를 자동으로 재검증 대상에 포함한다. forward-only 모드는 사람이 명시적으로 선택해야만 작동한다."
governs:
  - concept.round
  - concept.round-baseline
  - concept.regression
  - concept.convergence
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
  - "sdi-plugin/crates/core/src/round.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 핵심 가치 제안 중 하나는 "다음 작업 시작 시점에 이전 시나리오 전부를 자동 회귀 점검"이다(PRD §1.2). 이 자동성은 round 모드의 기본값이 `strict-regression`이어야 성립한다.

`strict-regression` 모드에서 Round가 active로 전환되면, `carry_over_results`가 R1..R(N-1)의 passing 시나리오 전체를 R(N)의 검증 대상으로 복사한다. "이전에 통과했다"는 사실이 암묵적 전제가 아니라 명시적 검증 항목이 된다. 이 검증을 통과해야만 R(N)의 passing으로 기록된다.

대안 모드인 `forward-only`는 retired 시나리오를 제외하고 동일 흐름을 따른다. 이 모드는 사람이 `/round start --mode forward-only`처럼 명시적으로 선택해야만 작동한다. 기본값으로 적용되지 않으며, 기본값 없이 묻지도 않는다.

## 깨지면 무슨 일이 일어나나

기본값이 `forward-only`가 되거나 사람이 모드를 선택하지 않아 회귀 검증이 생략된다면, 새 구현이 이전 시나리오를 깨뜨려도 R(N)이 passing으로 끝날 수 있다. "다음 작업이 이전 작업을 망치지 않는다"는 SDI의 핵심 보장이 사라진다. 특히 멀티 에이전트 병렬 실행 환경에서 한 에이전트의 작업이 다른 에이전트의 시나리오를 조용히 파괴하는 race 상황을 발견하지 못한다. 이는 D6의 설계 의도, 즉 "LLM이 자율 수행"을 "자율이지만 회귀 안전망 있음"으로 만드는 기반이 무너지는 것이다.

## 코드에서 어떻게 강제되나

데몬의 Round 활성화 경로(`crates/daemon/src/router/round.rs`)는 Round 행 생성 시 `mode` 컬럼이 지정되지 않으면 `strict_regression`을 기본값으로 사용한다. `carry_over_results` 함수는 `RoundMode::StrictRegression`일 때 이전 라운드의 passing 시나리오를 inner join으로 복사한다(unevaluated 시나리오는 제외). HOOK_ENFORCEMENT.md #3 인수기준과 `r2_auto_regression_carries_results` 테스트로 검증된다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D6 Round 모드 기본값 결정 엔트리) 연결 필요
