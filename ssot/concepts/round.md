---
id: concept.round
kind: Concept
title: Round(라운드)
definition: "플랜 안에서 시나리오를 검증하는 실행 단위. R1, R2, R3... 순서로 번호가 붙으며, R1은 신규 개발(forward-only), R2+는 회귀 검증(strict-regression 기본값)을 수행한다. planning → active → completed 생애주기를 따른다."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "라운드는 반드시 하나의 플랜에 속한다." }
  - { to: concept.task, type: relates-to, note: "라운드가 활성화되면 시나리오들이 태스크로 분해된다." }
  - { to: concept.scenario, type: relates-to, note: "라운드는 플랜의 시나리오들을 검증 대상으로 삼는다." }
  - { to: concept.round-baseline, type: relates-to, note: "라운드는 검증 시작 시점의 baseline(예: 통과 테스트 수)을 보존한다." }
  - { to: concept.regression, type: relates-to, note: "strict-regression 모드의 라운드는 이전 라운드의 passing 시나리오를 재검증한다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "라운드는 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: concept.disruption-review, type: relates-to, note: "pending 상태의 DisruptionReview가 있으면 라운드를 활성화할 수 없다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/round.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/014_round_baseline.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Round(라운드)는 플랜의 시나리오들을 실제로 코드와 대조해 검증하는 실행 단위다. 하나의 플랜은 여러 라운드를 순서대로 가질 수 있으며, 각 라운드는 round_number(1부터 시작)로 구분된다.

라운드의 생애주기는 세 단계다.

- **planning**: 생성은 되었으나 아직 에이전트가 실행을 시작하지 않음. 이 단계에서 라운드 설정을 조정할 수 있다.
- **active**: 에이전트가 시나리오를 분해해 태스크를 생성하고 검증을 수행 중. 한 플랜에서 동시에 active인 라운드가 여러 개 있을 수 있는지 여부는 따로 확인 필요.
- **completed**: 검증 완료.

라운드에는 두 가지 **검증 모드**가 있다.

- **strict-regression(기본값, D6)**: 이전 라운드에서 passing이었던 시나리오들도 이번 라운드에서 다시 검증한다. 회귀 감지가 목적이다.
- **forward-only(alias: additive)**: 이전 라운드 결과를 캐리오버하지 않고 신규 시나리오만 검증한다. R1(첫 번째 라운드)에서 주로 사용된다.

라운드 시작(planning→active) 시 **진행 중인 태스크 정책(D10)** 을 지정한다.

- **pause(기본값)**: 다른 라운드의 in_progress 태스크를 일시 동결.
- **abort**: in_progress 태스크를 즉시 취소.
- **continue-on-noimpact**: 새 라운드와 무관한(충돌하지 않는) in_progress 태스크는 계속 실행.

**disruption 정책(D9)** 은 라운드 단위로 지정된다. `needs-review`(기본값)와 `auto` 두 종류가 있으며, 어느 쪽이든 pending 상태의 DisruptionReview가 있으면 라운드를 활성화할 수 없다. `auto`는 LLM이 해소 방안을 자동 제안하는 것을 허용하되, 그 결정을 인간이 확인해야 함은 동일하다.

## 엔티티 (DB)

라운드 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜 참조.
- **식별**: 플랜 내 고유 short_code, round_number(1부터 순차 증가).
- **설정**: 검증 모드(strict-regression/forward-only), 진행 중 태스크 정책(pause/abort/continue-on-noimpact), disruption 정책(needs-review/auto).
- **상태**: planning/active/completed.
- **베이스라인**: baseline_json — 라운드 시작 시점에 기록된 검증 베이스라인(예: 통과 테스트 수, 커밋 SHA). 자유 형식 JSON.
- **패턴 출처(D23)**: produced_via_pattern_id.
- **타임스탬프**: 생성, 수정, 활성화(activated_at), 완료(completed_at) 시각.

## API 표면

라운드는 `/sdi round` 명령(/round 슬래시 커맨드 포함)으로 생성·활성화·완료한다. 활성화 시 pending DisruptionReview 존재 여부를 검사한다. baseline_json은 test-runner specialist가 라운드 활성화 직후 기록한다.

## 불변식

- **activate 게이트**: planning 상태일 때만 active로 전환 가능. pending DisruptionReview가 있으면 차단.
- **complete 게이트**: active 상태일 때만 completed로 전환 가능.
- **strict-regression 기본(D6)**: 명시적으로 forward-only를 지정하지 않으면 strict-regression 모드다.
- **D10 in-flight 정책**: active 전환 시 in-flight 태스크 처리 정책을 반드시 결정(기본: pause).

## 구현 위치 (provenance)

라운드 도메인 규칙과 열거형들은 `sdi-plugin/crates/core/src/round.rs`에 있다. baseline_json 컬럼은 마이그레이션 014에서 추가되었다.

## 미확정 (OPEN)

- [ ] OPEN: 한 플랜에서 동시에 active인 라운드가 여러 개 허용되는지 여부 — 코드에서 제약이 보이지 않으나 실제 운영 규칙 확인 필요.
- [ ] OPEN: forward-only 모드에서 이전 라운드 결과를 캐리오버하지 않는 구체적 동작(scenario_results 테이블 처리 방식) 확인 필요.
