---
id: endpoint.disruption-reviews-resolve
kind: Endpoint
title: "POST /disruption-reviews/:id/resolve"
definition: >
  사람이 pending 상태의 혼란 검토를 해결 처리하는 엔드포인트.
  해결 방식과 선택적 메모를 기록하고 상태를 resolved로 전환한다.
  해결되면 라운드 활성화를 막고 있던 needs-review 게이트가 해제된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
relatesTo:
  - to: concept.disruption-review
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

사람이 혼란 검토를 직접 검토하고 처리 방식을 결정한 뒤 호출하는 엔드포인트다. 호출하면 검토 상태가 `resolved`로 바뀌고, 이 플랜에 다른 미해결 검토가 없다면 라운드 활성화를 막던 needs-review 게이트가 해제된다. 해결 방식은 세 가지 중 하나로 기록된다: 시나리오를 폐기(`retired`), 시나리오를 수정(`edited`), 영향을 인지한 채 유지(`kept_impacted`). 해결 사실은 `disruption.resolved` SSE 이벤트로 발행된다.

## 요청 / 응답

**POST /disruption-reviews/:id/resolve**

경로 파라미터:

- `id`: 해결 처리할 혼란 검토의 식별자.

요청 본문:

- `resolution` (선택): 해결 방식. 다음 세 값 중 하나를 선택한다.
  - `retired`: 영향받는 시나리오를 폐기하기로 결정.
  - `edited`: 영향받는 시나리오를 수정하여 현황에 맞게 갱신하기로 결정.
  - `kept_impacted`: 영향을 인지하였으나 시나리오를 그대로 유지하기로 결정.
- `note` (선택): 해결 과정에서의 판단 근거나 추가 맥락을 자유 텍스트로 기술.

응답: 해결 처리된 혼란 검토 행. `status`가 `resolved`로 채워진 상태.

## 권한 / 제약

- 이 엔드포인트는 사람이 직접 호출하도록 설계되었다. 에이전트가 자동으로 호출하면 게이트의 목적이 훼손된다.
- 이미 `resolved` 상태인 검토에 재호출 시의 동작(멱등 처리 또는 오류) 명세에서 미확인.
- `resolution`이 선택 사항이므로 생략한 채 호출해도 상태가 전환된다.
- 해결 후 `disruption.resolved` SSE 이벤트가 발행된다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/disruption.rs`에서 유추됨. needs-review 게이트 해제 조건과 resolution 열거값은 POST /disruption-reviews 명세에서 참조.

## 미확정 (OPEN)

- 이미 resolved인 검토에 재호출 시 멱등 처리하는지 오류를 반환하는지 미확인.
- `resolution`을 생략했을 때 기본값으로 설정되는 값이 있는지, 또는 null로 저장되는지 미확인.
- 이 플랜에 다른 미해결 검토가 있을 때 게이트 해제 여부 판단 로직의 위치(이 엔드포인트 내부 vs. 라운드 활성화 시점) 미확인.
- SSE 이벤트 페이로드 구조 미확인.
