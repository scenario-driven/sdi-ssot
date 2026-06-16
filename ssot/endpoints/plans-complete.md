---
id: endpoint.plans-complete
kind: Endpoint
title: "POST /plans/:id/complete"
definition: 활성 상태의 플랜을 완료(completed) 상태로 전환한다. 모든 라운드가 끝나고 작업이 마무리되었을 때 호출한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
relatesTo:
  - to: concept.plan
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

활성 플랜을 완료 상태로 닫는 엔드포인트다. 해당 플랜에 속한 모든 라운드가 완료되고 구현 작업이 끝났을 때 호출한다. 완료된 플랜은 더 이상 라운드나 태스크를 추가할 수 없으며, 이력으로만 조회된다.

## 요청 / 응답

**POST /plans/:id/complete**
- 경로 파라미터 `id`: 완료 처리할 플랜의 고유 식별자.
- 요청 본문: 없음(body 불필요).
- 응답: 상태가 `completed`로 바뀐 플랜 객체.
- 완료 성공 시 `plan.completed` SSE 이벤트를 발행한다.

## 권한 / 제약

- 활성(`active`) 상태의 플랜에만 호출할 수 있다. 초안이나 이미 완료된 플랜에 호출하면 오류를 반환한다.
- 완료 전에 미완료 라운드나 열린 태스크가 남아 있는 경우의 처리 방침은 미확정이다(강제 완료 허용 여부 확인 필요).
- 완료 후에는 해당 플랜의 상태를 되돌릴 수 없다.

## provenance

sdi-plugin 플랜 라우터(`plan.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. 상태 전환 조건과 SSE 이벤트명은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- 미완료 라운드나 열린 태스크가 있을 때 완료 호출을 차단하는지, 아니면 경고만 하고 진행하는지 미확인.
- `plan.completed` SSE 이벤트의 페이로드 스키마 미확인.
- 완료된 플랜에 속한 시나리오·요구사항의 상태가 자동으로 바뀌는지 미확인.
- 완료 권한(특정 역할 또는 소유자만 가능 여부) 미확인.
