---
id: concept.round-baseline
kind: Concept
title: Round Baseline(라운드 베이스라인)
definition: "라운드가 시작될 때 test-runner specialist가 기록하는 검증 시작점 스냅샷. 통과 테스트 수, 커밋 SHA 등을 자유 형식 JSON으로 저장하며, 이후 라운드 실행 중 회귀(regression)를 탐지하는 기준점으로 사용된다."
relatesTo:
  - { to: concept.round, type: belongs-to, note: "베이스라인은 라운드의 baseline_json 필드에 저장된다." }
  - { to: concept.regression, type: relates-to, note: "베이스라인과 현재 상태를 비교해 회귀를 감지한다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/db/src/migrations/014_round_baseline.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Round Baseline은 "이 라운드가 시작되었을 때 시스템이 어떤 상태였는가"를 기록한 스냅샷이다. 주된 용도는 회귀 탐지다. 라운드 실행 중 또는 완료 후 현재 상태와 베이스라인을 비교해 "이전보다 테스트가 줄었는가", "특정 커밋 이후 변화가 있는가" 등을 확인할 수 있다.

베이스라인의 형식은 test-runner specialist가 자유롭게 정의한다. 시스템(daemon)은 형식을 강제하지 않는다. 예를 들어 `{"green_tests": 412, "ref": "abc1234"}` 형태로 기록될 수 있다.

베이스라인은 nullable이다. 기록되지 않은 라운드는 회귀 탐지에 베이스라인 비교를 사용할 수 없다.

## 엔티티 (DB)

라운드 테이블(rounds)에 `baseline_json` 컬럼으로 저장된다. TEXT 타입이며 nullable. 마이그레이션 014에서 추가되었다.

## API 표면

test-runner specialist가 라운드 활성화 직후 baseline을 기록한다. `sdi task brief` 명령이 이 베이스라인을 반환해 에이전트가 현재 상태와 비교할 수 있다.

## 불변식

- **형식 자유**: daemon은 baseline_json의 JSON 형식을 강제하지 않는다. 형식 계약은 test-runner specialist가 정한다.
- **nullable**: 베이스라인이 없는 라운드도 유효하다.

## 구현 위치 (provenance)

`baseline_json` 컬럼 추가는 `sdi-plugin/crates/db/src/migrations/014_round_baseline.sql`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: baseline_json을 기록하는 정확한 API 엔드포인트 또는 MCP 도구 이름 확인 필요.
- [ ] OPEN: baseline을 기록하는 주체가 test-runner specialist로 고정되어 있는지 또는 다른 에이전트도 기록할 수 있는지 확인 필요.
