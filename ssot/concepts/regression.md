---
id: concept.regression
kind: Concept
title: Regression(회귀)
definition: "이전 라운드에서 passing이었던 시나리오가 현재 라운드에서 failing 또는 impacted로 판정되는 현상. strict-regression 모드(D6 기본값)의 라운드는 이전 라운드의 passing 시나리오들을 현재 라운드에서 다시 검증해 회귀를 탐지한다."
relatesTo:
  - { to: concept.round, type: relates-to, note: "strict-regression 모드 라운드가 회귀 탐지를 수행한다." }
  - { to: concept.scenario, type: relates-to, note: "회귀는 시나리오 단위로 판정된다." }
  - { to: concept.round-baseline, type: relates-to, note: "라운드 베이스라인과 현재 상태를 비교해 회귀를 수량화할 수 있다." }
  - { to: concept.task-evidence, type: relates-to, note: "태스크 증거의 result=impacted는 회귀 신호다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/round.rs
  - sdi-plugin/crates/core/src/scenario.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Regression(회귀)은 소프트웨어가 "이전에 잘 작동하던 것이 지금은 작동하지 않게 된" 상태다. SDI에서 회귀는 시나리오 단위로 정의된다.

SDI의 회귀 탐지 흐름은 다음과 같다.

1. R1 라운드에서 시나리오 S1이 passing으로 판정된다.
2. R2 라운드(strict-regression 모드, 기본값)가 시작되면 S1이 carry-over 검증 대상이 된다.
3. R2에서 S1을 검증한 결과 failing 또는 impacted가 나오면 "회귀 발생"이다.

`ScenarioResult.impacted`는 "이번 변경이 이 시나리오에 영향을 미쳤음"을 의미하며 회귀의 신호다. `failing`은 명확한 실패다.

라운드 베이스라인(`baseline_json`)은 회귀를 수량화하는 참조점이다. 예를 들어 "라운드 시작 시 통과 테스트 412개"를 기록해 두면, 라운드 종료 후 통과 테스트 수와 비교해 얼마나 줄었는지 알 수 있다.

regression-runner specialist가 이전 라운드의 passing 시나리오 목록을 가져와 현재 라운드에서 재검증하는 역할을 담당한다.

## 엔티티 (DB)

Regression은 독립 엔티티가 아니다. `scenario_results` 테이블에 라운드별 시나리오 판정 결과가 저장된다(마이그레이션 001에서 정의). round_baseline은 rounds 테이블의 `baseline_json` 컬럼에 저장된다(마이그레이션 014).

## API 표면

regression-runner specialist가 daemon에서 이전 라운드의 passing 결과를 조회하고, 현재 라운드에서 같은 시나리오들을 재검증해 태스크 증거로 기록한다.

## 불변식

- **strict-regression 기본(D6)**: 명시적으로 forward-only를 지정하지 않으면 이전 라운드 결과를 carry-over해 재검증한다.
- **결과값 의미**: impacted와 failing은 모두 회귀 신호. passing은 회귀 없음.

## 구현 위치 (provenance)

`ScenarioResult` 열거형과 `RoundMode::StrictRegression`은 `sdi-plugin/crates/core/src/scenario.rs`와 `sdi-plugin/crates/core/src/round.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: R1이 항상 forward-only여야 하는지, 아니면 R1도 strict-regression으로 설정 가능한지 — PRD에서 "R1 = new, R2+ = regression"이라 설명하나 기술적 강제 여부 확인 필요.
