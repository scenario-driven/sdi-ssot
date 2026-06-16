---
id: endpoint.rounds-baseline
kind: Endpoint
title: "POST /rounds/:id/baseline, GET /rounds/:id/baseline"
definition: 라운드 시작 전 시스템 상태를 JSON 스냅샷으로 저장하고 조회하는 엔드포인트. 라운드 종료 후 회귀 비교를 위한 기준점(베이스라인) 역할을 한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.round-baseline
    type: mutates
    note: ""
  - to: concept.round
    type: reads
    note: ""
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 라운드의 사전 시스템 상태를 JSON 스냅샷으로 저장하거나 조회하는 엔드포인트다. 라운드가 시작되기 전에 POST 로 현재 시스템 상태를 기록해 두고, 라운드 완료 후 회귀 분석 시 GET 으로 그 기준점(베이스라인)을 불러와 비교한다.

## 요청 / 응답

**POST /rounds/:id/baseline** — 베이스라인 스냅샷 저장
- 경로 파라미터: `id`(라운드 ID, 필수)
- 요청 본문: `baseline_json`(임의 JSON, 필수) — 저장할 시스템 상태 스냅샷
- 응답: 저장 성공 확인 (저장된 베이스라인 객체 또는 성공 메시지)

**GET /rounds/:id/baseline** — 베이스라인 스냅샷 조회
- 경로 파라미터: `id`(라운드 ID, 필수)
- 응답: 저장된 베이스라인을 파싱된 JSON 으로 반환. 베이스라인이 없으면 null 반환.

## 권한 / 제약

- 존재하지 않는 라운드 ID 를 지정하면 404 를 반환한다.
- 베이스라인은 라운드 당 하나만 존재한다. POST 를 두 번 호출하면 덮어쓰는지, 오류를 반환하는지 현재 명세상 불명확하다.
- `baseline_json` 의 구조는 제한하지 않으며 임의의 JSON 을 수용한다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 베이스라인 저장/조회 기능은 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- 베이스라인이 이미 존재하는 상태에서 POST 를 재호출할 때의 동작(덮어쓰기 vs 409 오류) 미확인.
- 베이스라인 저장 시 SSE 이벤트가 발행되는지 미확인.
- `baseline_json` 의 권장 스키마(표준 형식) 가 있는지, 또는 순수하게 자유 형식인지 미확인.
