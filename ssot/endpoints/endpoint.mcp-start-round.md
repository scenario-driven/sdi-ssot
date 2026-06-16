---
id: endpoint.mcp-start-round
kind: Endpoint
title: MCP start_round — 라운드 활성화
definition: LLM 에이전트 또는 운영자가 플랜의 특정 라운드를 활성화하고 이전 라운드의 통과 결과를 자동 이월하는 MCP 쓰기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/write.rs
relatesTo:
  - to: concept.round
    type: mutates
  - to: concept.regression
    type: reads
  - to: concept.plan
    type: reads
  - to: domain.round-execution
    type: belongs-to
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`start_round` 는 SDI MCP 서버의 쓰기 도구로, LLM 에이전트나 운영자가 플랜의 다음 라운드를 활성화할 때 사용한다. 데몬의 `POST /plans/:id/rounds/:round_id/activate` 엔드포인트로 위임된다.

SDI 에서 라운드는 반복 검증 주기다. R1(첫 라운드)에서는 모든 시나리오를 처음 구현하고, R2 이후부터는 새 기능 개발과 함께 이전 라운드에서 통과한 시나리오들이 자동으로 이월되어 회귀 검증을 수행한다. `start_round` 를 호출하면 D6 엄격 회귀 정책에 따라 이전 라운드에서 `passing` 상태였던 모든 시나리오 결과가 새 라운드의 회귀 베이스라인으로 즉시 등록된다. 에이전트는 이 베이스라인을 무너뜨리지 않으면서 새 기능을 구현해야 한다.

## 요청 / 응답

**입력이 의미하는 것**

플랜 ID와 라운드 ID를 함께 지정한다. 플랜 ID는 어느 계획의 라운드인지를, 라운드 ID는 활성화할 특정 라운드를 식별한다. 라운드는 플랜 승인 후 생성되며, 이 도구는 이미 생성된 라운드를 `planning` 상태에서 `active` 상태로 전환하는 역할만 한다.

**출력이 의미하는 것**

응답은 활성화된 라운드의 정보와 함께 `carried_results`(이월된 시나리오 결과 수)를 반환한다. 이월 수는 이 라운드부터 회귀 검증 대상이 되는 시나리오의 기준 개수를 나타낸다. R1 라운드에서는 이전 라운드가 없으므로 `carried_results`가 0이다.

## 권한 / 제약

대상 라운드가 반드시 `planning` 상태여야 한다. 이미 `active` 또는 `completed` 상태인 라운드를 재활성화하려는 시도는 데몬이 거부한다.

플랜 내에 이미 `active` 상태인 다른 라운드가 존재하면 새 라운드를 활성화할 수 없다. 한 플랜에는 동시에 하나의 active 라운드만 허용된다.

D6 이월 처리는 이 도구 호출 시점에 원자적으로 수행된다. `carried_results`로 이월된 시나리오들은 즉시 회귀 베이스라인으로 등록되며, 에이전트는 이후 `update_task_evidence` 호출 시 이 시나리오들을 `passing` 상태로 유지해야 한다. `failing`이나 `impacted`로 전환되면 회귀 감지가 발동된다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/write.rs` 구현 분석에서 추론
- 위임 엔드포인트: `POST /plans/:id/rounds/:round_id/activate` (데몬 REST API)
- 관련 설계 결정: D6 (엄격 회귀 정책 — 이전 라운드 통과 결과 자동 이월)

## 미확정 (OPEN)

- 라운드를 `planning` 상태로 생성하는 도구가 MCP 레이어에 별도로 있는지, CLI 경로만 지원되는지 미확인
- `carried_results` 이월 범위가 `passing` 시나리오만인지, `impacted` 시나리오도 포함되는지 D6 정책 세부 확인 필요
- 단일 active 라운드 제약의 예외 케이스(긴급 롤백 등) 존재 여부가 불명확
- 라운드 ID 없이 "다음 라운드 자동 생성 후 활성화"하는 단축 경로가 있는지 미확인
