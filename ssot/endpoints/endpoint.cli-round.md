---
id: endpoint.cli-round
kind: Endpoint
title: "CLI `sdi round`"
definition: "D6 라운드 생애주기 및 회귀 결과 관리 — 라운드 생성·활성화·완료·베이스라인·결과 기록"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/round.rs"
  - "sdi-plugin/plugin/commands/round.md"
relatesTo:
  - to: "concept.round"
    type: "mutates"
  - to: "concept.regression"
    type: "mutates"
  - to: "concept.plan"
    type: "reads"
  - to: "domain.round-execution"
    type: "belongs-to"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`sdi round` 는 SDI 시스템에서 반복 실행 단위인 라운드(Round)의 생애주기와 검증 결과를 관리하는 CLI 명령 그룹이다. 라운드는 플랜 내에서 확정된 시나리오 집합을 대상으로 한 번의 완전한 실행 주기를 나타낸다.

D6 엄격 회귀 정책이 라운드의 핵심 동작 원칙이다. 새 라운드를 활성화(activate)할 때, 이전 라운드에서 통과한 시나리오 결과가 현재 라운드에 자동으로 이월(carried_results)된다. 이는 이전 라운드에서 검증된 동작이 새 구현에 의해 망가지지 않았음을 반드시 재확인해야 한다는 원칙을 구조로 강제하는 메커니즘이다.

D7 및 D10 규칙에 따른 기본값이 라운드 생성 시 자동 적용된다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: 플랜에 귀속된 새 라운드를 생성한다. 라운드 번호는 플랜 내에서 자동으로 순차 증가한다. 생성 직후 상태는 준비(pending) 상태다.

**활성화(activate)**: 라운드를 실행 가능한 active 상태로 전환한다. 이 시점에 D6 이월 정책이 적용되어, 이전 라운드의 통과 결과가 현재 라운드의 기준으로 복사된다.

**완료(complete)**: 활성 라운드를 completed 상태로 전환한다. 미검증 시나리오가 남아 있으면 경고가 표시된다.

**베이스라인(baseline)**: 현재 라운드의 검증 결과를 기준 베이스라인으로 설정한다. 이후 라운드의 회귀 비교 기준점이 된다.

**결과 기록(result / results)**: 개별 시나리오에 대한 검증 결과(통과/실패/건너뜀)를 라운드에 기록한다. `results`는 여러 시나리오 결과를 일괄 기록할 때 사용한다. 각 결과에는 검증 방법, 근거, 실행 에이전트 정보를 함께 기록할 수 있다.

**활성 라운드 조회(active)**: 현재 플랜에서 active 상태인 라운드를 빠르게 조회한다.

## 권한 / 제약

- 한 플랜에서 동시에 active 상태인 라운드는 하나뿐이어야 한다. 이전 라운드가 completed 상태가 되어야 새 라운드를 activate할 수 있다.
- D6 이월 정책: R2 이후의 모든 라운드는 이전 라운드 통과 결과를 이월하며, 이 결과들은 현재 라운드에서도 재검증 대상이다.
- `result` 기록은 active 상태의 라운드에만 허용된다. completed 라운드의 결과는 수정할 수 없다.
- 첫 번째 라운드(R1)는 이전 라운드가 없으므로 이월 결과 없이 시작한다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/round.rs` 및 `sdi-plugin/plugin/commands/round.md` 파일 구조와 SDI PRD D6/D7/D10 설계 규칙을 근거로 추론되었다. `carried_results` 이월 메커니즘은 설계 명세 기반 추론이며 코드 구현 세부 사항은 별도 검증이 필요하다.

## 미확정 (OPEN)

- `baseline` 서브커맨드와 D6 이월의 정확한 관계가 명확하지 않다. 베이스라인이 이월 기준인지, 별도의 독립적 개념인지 확인이 필요하다.
- `activate` 시 이전 라운드가 completed 상태가 아닐 경우 강제 활성화를 허용하는지 여부가 불명확하다.
- `result` 기록 시 에이전트 신원 검증(누가 결과를 기록했는지)의 강제 수준이 확인되지 않았다.
