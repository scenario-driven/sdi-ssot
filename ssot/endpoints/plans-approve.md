---
id: endpoint.plans-approve
kind: Endpoint
title: "POST /plans/:id/approve"
definition: 플랜을 초안(draft) 상태에서 활성(active) 상태로 전환한다. D8 게이트: 확정된 GWT 시나리오가 1개 이상 없으면 거부한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
relatesTo:
  - to: concept.plan
    type: mutates
  - to: concept.scenario
    type: reads
    note: "D8 게이트: 확정된 시나리오가 1개 이상 있어야 승인이 통과된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

플랜 승인 엔드포인트다. 초안 상태의 플랜을 활성 상태로 바꾸는 단일 동작을 수행한다. 플랜이 활성 상태가 되어야만 라운드 생성과 태스크 분해가 가능하다. SDI의 D8 게이트 규칙을 강제하는 지점으로, 시나리오 없이 플랜을 진행하는 것을 방지한다.

## 요청 / 응답

**POST /plans/:id/approve**
- 경로 파라미터 `id`: 승인할 플랜의 고유 식별자.
- 요청 본문: 없음(body 불필요).
- 응답: 상태가 `active`로 바뀐 플랜 객체.
- D8 게이트 실패(확정된 시나리오 0개)이면 요청을 거부하고 오류 응답을 반환한다.
- 승인 성공 시 `plan.approved` SSE 이벤트를 발행한다.

## 권한 / 제약

- **D8 게이트**: 해당 플랜에 속하는 확정(confirmed) 상태의 GWT 시나리오가 1개 이상 존재해야 한다. 이 조건이 충족되지 않으면 서버가 승인을 거부한다.
- 이미 활성 또는 완료 상태인 플랜에는 호출할 수 없다(초안 상태에서만 유효).
- 승인 후에는 라운드 생성(`POST /rounds`)과 태스크 분해가 허용된다.

## provenance

sdi-plugin 플랜 라우터(`plan.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. D8 게이트 조건(≥1 confirmed scenario)은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- D8 게이트 실패 시 반환되는 HTTP 상태 코드(400 vs 422 vs 409) 미확인.
- 복수의 확정 시나리오 중 일부가 특정 상태(예: deprecated)이면 카운트에서 제외하는지 여부 미확인.
- `plan.approved` SSE 이벤트의 페이로드 스키마 미확인.
- 승인 권한(특정 역할 또는 소유자만 가능 여부) 미확인.
