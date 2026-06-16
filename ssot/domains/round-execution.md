---
id: domain.round-execution
kind: Domain
title: 라운드 실행 (Round Execution)
purpose: "R1(신규 개발)과 R2+(회귀 포함) 라운드를 관리하며, 이전 라운드에서 passing이었던 모든 시나리오를 자동으로 회귀 재실행해 새 작업이 기존 동작을 깨뜨리지 않음을 보장한다."
definition: "Round 엔티티의 생성·활성화·실행·완료를 담당하는 경계. R1은 신규 개발, R2+는 회귀 검증이지만 같은 엔진으로 처리된다(D7). 기본 모드는 strict-regression — 이전 라운드의 모든 passing 시나리오를 재실행한다. in-flight 태스크가 있으면 기본 pause하고 처리 방식을 선택할 수 있다(D10)."
servesPersona:
  - persona.coding-agent
  - persona.subagent
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: 라운드가 시나리오를 집행 단위로 삼는다. R2+는 이전 라운드 passing 시나리오를 자동으로 재실행한다.
  - to: domain.task-decomposition
    type: relates-to
    note: 라운드 안에서 태스크가 생성·실행된다. 라운드가 활성화되어야 태스크를 만들 수 있다.
  - to: domain.planning
    type: relates-to
    note: 라운드는 플랜에 귀속된다. 플랜이 active여야 라운드를 시작할 수 있다.
  - to: domain.disruption-review
    type: relates-to
    note: 라운드 시작 시 in-flight 태스크가 있으면 disruption-analyst가 영향 범위를 점검한다.
  - to: concept.round
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.round-baseline
    type: relates-to
    note: R2+ 라운드가 회귀 기준으로 삼는 이전 라운드의 passing 결과 집합이다.
  - to: concept.regression
    type: relates-to
    note: strict-regression 모드가 이 도메인의 핵심 보장이다.
  - to: concept.convergence
    type: relates-to
    note: 라운드 안의 모든 시나리오가 passing이 되면 라운드가 수렴(converged)한다.
governedBy: []
realizedBy: []
impacts:
  - domain.scenario-management
  - domain.disruption-review
implementedIn:
  - "sdi-plugin/crates/core/src/round.rs"
  - "sdi-plugin/crates/daemon/src/router/round.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI가 Clawket 대비 제공하는 핵심 차별점을 구현한다 — 자동 회귀 검증. 새 작업을 시작할 때마다 이전에 통과했던 모든 시나리오를 자동으로 다시 실행해, 새 코드가 기존 동작을 깨뜨리지 않았음을 기계적으로 확인한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **라운드 번호와 의미**: R1은 플랜의 첫 구현 라운드다. R2+는 이전 라운드의 passing 시나리오를 기준(baseline)으로 삼아 회귀 재실행을 포함한다. 두 흐름이 같은 엔진을 쓴다(D7) — R1도 R2+도 "시나리오 집합을 처리하고 판정을 기록한다"는 동일한 구조다.

- **strict-regression 모드(기본)**: R2+에서 이전 라운드의 모든 passing 시나리오를 재실행한다. 중간에 retired로 명시 해제된 것은 제외한다. 회귀가 발견되면(이전에 passing이었는데 failing) 즉시 보고된다.

- **forward-only 모드(선택)**: 빌더가 명시적으로 선택해야 활성화된다. 이전 시나리오 중 retired만 제외하고 재실행하되, baseline을 더 넓게 잡는다.

- **in-flight 태스크 처리(D10)**: 진행 중 태스크가 있는 상태에서 새 라운드를 시작하려 하면 기본적으로 일시 중지한다. 빌더가 `--abort`(중단), `--continue-on-noimpact`(영향 없으면 계속) 플래그로 처리 방식을 선택할 수 있다.

- **라운드 상태**: `planning → active → completed`. 이전 라운드(R-N)가 active인 동안 R-(N+1)은 시작할 수 없다.

포함되지 않는 것: 시나리오 GWT 작성(domain.scenario-management), 태스크 개별 생성(domain.task-decomposition), 파괴 분석(domain.disruption-review).

## 기능

- 플랜에 새 라운드를 생성한다(R1이면 신규, R2+이면 이전 라운드 baseline 로드).
- 라운드를 활성화하고 태스크 생성·실행을 허용한다.
- R2+ 모드에서 이전 passing 시나리오 목록을 자동으로 집행 대상에 포함한다.
- 라운드 내 모든 시나리오의 판정이 완료되면 라운드를 completed로 전환한다.
- 라운드 결과(시나리오별 판정·증거)를 조회한다.

## 시스템 흐름

플랜이 active 상태에서 코딩 에이전트가 라운드를 시작한다. R1이면 플랜의 confirmed 시나리오 전체가 집행 대상이 된다. R2+이면 이전 라운드의 passing 시나리오를 baseline으로 불러와 자동 포함한다. scenario-decomposer가 태스크를 생성하고, impl-coder가 구현하고, test-runner가 판정과 증거를 제출하고, regression-runner가 baseline 시나리오를 재실행한다. 모든 시나리오가 처리되면 라운드가 completed로 전환되고, 다음 R+(N+1)의 baseline이 된다.

## 다른 도메인과의 관계

라운드는 시나리오와 태스크를 하나의 실행 단위로 묶어 진행 상황을 추적하는 그릇이다. 플랜이 "무엇을 만들 것인가"라면, 라운드는 "이번에 무엇을 만들고 무엇을 검증할 것인가"다. disruption-review 도메인과 밀접하게 연결되어, 라운드 시작 시점에 기존 시나리오에 미치는 영향을 검토한다.

## 미확정 (OPEN)
- [ ] OPEN: R-(N)이 active인 동안 R-(N+1) 시작을 완전히 차단하는지, 아니면 경고 수준인지 현재 구현에서 확인 필요.
