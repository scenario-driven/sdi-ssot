---
id: endpoint.rounds-activate
kind: Endpoint
title: "POST /rounds/:id/activate"
definition: 라운드를 계획(planning) 상태에서 활성(active) 상태로 전환하는 엔드포인트. 복수의 사전 조건 게이트를 통과해야 전환이 허용된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
    type: mutates
  - to: concept.regression
    type: mutates
    note: "D6: 이전 완료 라운드의 시나리오 결과를 회귀 기준선으로 복사"
  - to: concept.disruption-review
    type: reads
    note: "미완료 disruption-review가 존재하면 활성화가 차단됨"
  - to: concept.task
    type: mutates
    note: "in_flight_policy에 따라 진행 중 태스크가 일시 정지·중단·유지됨"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

라운드를 `planning` 상태에서 `active` 상태로 전환한다. 활성화 전에 세 가지 게이트가 순서대로 검사된다.

**게이트 1 — disruption-review 대기 차단**: 해당 플랜에 아직 처리되지 않은 disruption-review(충돌 검토)가 하나라도 존재하면 활성화가 거부된다. 영향을 받은 시나리오를 사람이 먼저 검토하고 해소해야 한다.

**게이트 2 — 진행 중 태스크 처리**: 활성화 시점에 이미 진행 중인 태스크가 있을 경우, 라운드의 `in_flight_policy` 설정에 따라 일시 정지(`pause`), 강제 중단(`abort`), 또는 그대로 유지(`continue`) 처리된다.

**게이트 3 — D6 strict-regression 기준선 복사**: `mode`가 `strict-regression`이고 이전에 완료된 라운드가 존재하면, 그 라운드의 시나리오 검증 결과 전체를 현재 라운드의 회귀 기준선으로 복사한다. 이를 통해 새 라운드에서 이미 통과했던 시나리오가 퇴행하지 않는지 자동으로 추적할 수 있다.

## 요청 / 응답

**POST /rounds/:id/activate**

요청 본문: 없음(경로 파라미터만 사용).

응답(JSON):

- `round`: 상태가 `active`로 전환된 라운드 객체.
- `carried_results_count`: 이전 라운드에서 복사된 시나리오 결과 수(D6 적용 시).
- `scenarios_needing_verification`: 이번 라운드에서 검증이 필요한 시나리오 목록.
- `in_flight_task_action`: 진행 중 태스크에 실제로 취해진 조치 요약(`paused`, `aborted`, `continued`).

## 권한 / 제약

- 이미 `active` 또는 `completed` 상태인 라운드에 다시 활성화를 요청하면 오류를 반환한다.
- 미완료 disruption-review가 존재하면 게이트 1에서 요청이 거부되며, 해소해야 할 disruption-review 목록이 응답에 포함된다.
- 한 플랜에 동시에 `active` 상태인 라운드는 최대 하나다.

## provenance

D6(strict-regression 기준선 복사), D9(disruption-review 필수 해소 정책)에 근거한다. `in_flight_policy` 처리 방식은 라운드 생성 시 설정된 값을 그대로 따른다.

## 미확정 (OPEN)

- 게이트 1에서 거부될 때 응답 형식(오류 코드, 미해소 disruption-review 목록 포함 여부) 세부 사항 미확인.
- 기준선 복사 대상이 완료된 직전 라운드인지, 아니면 가장 최근 완료 라운드 전체인지 미확인.
- `scenarios_needing_verification` 목록의 필터 기준(기준선 대비 변경된 것만 포함하는지 여부) 미확인.
