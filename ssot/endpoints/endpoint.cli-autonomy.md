---
id: endpoint.cli-autonomy
kind: Endpoint
title: "CLI `sdi autonomy`"
definition: "D14/D17/D18 자율성 정책 설정·조회·circuit-breaker 관리"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/autonomy.rs"
  - "sdi-plugin/plugin/commands/autonomy.md"
relatesTo:
  - to: "concept.autonomy-policy"
    type: "mutates"
  - to: "concept.plan"
    type: "reads"
  - to: "domain.autonomy"
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

`sdi autonomy` 는 SDI 시스템에서 에이전트 자율 실행 수준을 제어하는 자율성 정책(Autonomy Policy)을 관리하는 CLI 명령 그룹이다. 자율성 정책은 에이전트가 사람의 확인 없이 얼마나 자율적으로 의사결정하고 실행할 수 있는지를 레벨로 표현한다.

자율성 레벨은 L3(낮은 자율성, 주요 결정마다 사람 확인 필요)부터 L5(완전 자율, 사람 개입 최소화)까지 정의된다. D17 강제 L4 불변식이 핵심 제약으로, 특정 조건(예: 아키텍처 변경, 데이터 스키마 변경)에서는 에이전트가 설정 레벨에 관계없이 L4 이하로 내려갈 수 없다. D18 규칙은 정책 해석 우선순위를 정의한다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**설정(set)**: 특정 플랜 또는 전역 수준에서 자율성 레벨을 설정한다. 레벨 값(L3/L4/L5)과 적용 범위(플랜 ID 또는 전역)를 입력받는다. D17 불변식에 의해 일부 범위에서는 L3 설정이 거부될 수 있다.

**조회(get)**: 현재 적용 중인 자율성 정책을 조회한다. 플랜별 정책과 전역 기본값이 함께 표시되며, 실제 적용되는 유효 레벨과 그 결정 근거(플랜 정책인지 전역 기본값인지)가 명시된다.

**목록 조회(list)**: 모든 플랜에 설정된 자율성 정책을 일괄 조회한다. 정책이 없는 플랜은 전역 기본값이 적용됨을 표시한다.

**circuit-breaker 관리(circuit_breaker)**: 에이전트의 자율 실행을 일시 정지하거나 재개한다. circuit-breaker는 예상치 못한 동작 패턴이 감지되거나 긴급 개입이 필요할 때 즉시 자율 실행을 차단하는 안전 장치다. `trip`으로 차단하고 `reset`으로 재개한다.

## 권한 / 제약

- D17 강제 L4 불변식: 아키텍처 결정, 데이터 스키마 변경 등 고위험 작업 범주에서는 에이전트가 L4 미만으로 내려갈 수 없다. `set L3` 요청이 이 조건에 해당하면 거부된다.
- D18 정책 해석 순서: 플랜 레벨 정책이 전역 기본값보다 우선 적용된다. 플랜 정책이 없으면 전역 기본값을 사용한다.
- circuit-breaker가 활성화된 상태에서는 자율성 레벨과 관계없이 에이전트 자율 실행이 차단된다.
- 자율성 레벨 변경은 이미 실행 중인 에이전트 작업에 즉시 영향을 미치는지, 다음 작업부터 적용되는지 정책이 별도로 존재할 수 있다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/autonomy.rs` 및 `sdi-plugin/plugin/commands/autonomy.md` 파일 구조와 SDI PRD D14/D17/D18 설계 규칙을 근거로 추론되었다. circuit-breaker 서브커맨드 인터페이스(trip/reset)는 설계 의도 기반 추론이다.

## 미확정 (OPEN)

- D17 강제 L4 불변식의 구체적 적용 조건 목록(어떤 작업 유형이 강제 L4 대상인지)이 코드 레벨에서 확인되지 않았다.
- circuit-breaker 활성화 이벤트가 자동으로 감지·트리거되는지(자동 trip), 아니면 항상 수동으로 조작해야 하는지 불확실하다.
- L3/L4/L5 각 레벨에서 에이전트가 자율적으로 결정할 수 있는 행동의 구체적 범위가 문서화되어 있지 않다.
- `circuit_breaker reset` 후 이전에 차단된 대기 중 작업들의 처리 방식(재실행, 취소, 사람 확인 후 재실행)이 명확하지 않다.
