---
id: endpoint.rounds-complete
kind: Endpoint
title: "POST /rounds/:id/complete"
definition: 활성 상태인 라운드를 완료(completed) 상태로 전환하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/round.rs"
relatesTo:
  - to: concept.round
    type: mutates
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`active` 상태인 라운드를 `completed` 상태로 전환한다. 라운드 안의 모든 시나리오가 검증되었고 진행 중인 태스크가 없을 때 호출한다.

라운드가 완료되면 `round.completed` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /rounds/:id/complete**

요청 본문: 없음(경로 파라미터만 사용).

응답(JSON):

- 상태가 `completed`로 전환된 라운드 객체. 완료 시각(`completed_at`) 포함.

## 권한 / 제약

- `active` 상태가 아닌 라운드(이미 `completed`이거나 아직 `planning`)에는 오류를 반환한다.
- 라운드 완료가 모든 시나리오의 `passing` 상태를 강제하지는 않는다. 어떤 결과 상태가 완료 허용 조건인지는 제품 정책에서 결정하며, 데몬은 이를 강제하는 게이트를 두지 않는다.

## provenance

라운드 완료 후 SSE 이벤트(`round.completed`)를 발행하는 것은 CLI 및 대시보드가 라운드 상태 변화를 실시간으로 반영하기 위해 구독하는 흐름과 연결된다. 완료된 라운드의 결과는 다음 라운드 활성화 시 D6 기준선 복사의 소스가 된다.

## 미확정 (OPEN)

- 진행 중인 태스크가 남아 있을 때 강제 완료를 허용하는지, 아니면 거부하는지 미확인.
- `round.completed` SSE 이벤트의 페이로드 스키마 세부 사항 미확인.
- 완료 전에 모든 시나리오에 verdict가 기록되어 있어야 하는지 여부(소프트 체크 vs. 하드 게이트) 미확인.
