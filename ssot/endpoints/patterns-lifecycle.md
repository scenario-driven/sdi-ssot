---
id: endpoint.patterns-lifecycle
kind: Endpoint
title: "PATCH /patterns/:id/lifecycle"
definition: 협업 패턴의 라이프사이클 상태를 전환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/pattern.rs"
relatesTo:
  - to: concept.collaboration-pattern
    type: mutates
    note: "패턴의 lifecycle 필드를 대상 상태로 갱신한다"
  - to: concept.pattern-lifecycle
    type: mutates
    note: "유효한 전이 경로만 허용하며 종료 상태에서의 추가 전이는 차단한다"
  - to: concept.convergence
    type: mutates
    note: "converged 종료 상태로의 전환이 패턴 완료를 기록한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

협업 패턴의 라이프사이클을 다음 상태로 전환한다. 허용된 전이 경로는 다음과 같다.

- `pending → active`: 패턴을 활성화한다. 이 전환 시점에 형상 유효성 검사(D27)가 실행된다.
- `active → converged`: 패턴이 의도한 결과로 완료되었음을 기록한다.
- `active → dissensus`: 패턴이 합의 없이 종료되었음을 기록한다.
- `active → aborted`: 패턴이 중단되었음을 기록한다.

종료 상태(`converged`, `dissensus`, `aborted`)에 도달한 패턴은 더 이상 전환이 불가능하다.

## 요청 / 응답

**PATCH /patterns/:id/lifecycle**

경로 파라미터:

- `id` — 전환할 패턴의 식별자. 필수.

요청 본문:

- `to` — 전환할 목표 라이프사이클 상태 문자열. 필수. (`active`, `converged`, `dissensus`, `aborted` 중 하나.)
- `reason` — 전환 이유 설명 텍스트. 선택.

`pending → active` 전환 시 형상 유효성 검사(D27)가 먼저 실행된다. `workflow` 패턴에 `steps` 가 없거나, `graph` 패턴에 `reviewers` 가 없는 등 유형에 필수적인 형상 필드가 누락된 경우 400 오류를 반환하며 전환을 거부한다.

종료 상태(`converged`, `dissensus`, `aborted`)로 전환하면 데몬이 `decided_at` 을 현재 시각으로 자동 기록한다. 요청 본문의 `reason` 이 있으면 `decided_reason` 에 함께 저장된다.

성공 응답: 갱신된 패턴 객체. `pattern_lifecycle` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

## 권한 / 제약

- 이미 종료 상태인 패턴에 전환 요청을 보내면 409 오류를 반환한다.
- 허용되지 않는 전이 경로(예: `pending → converged`)를 요청하면 400 오류를 반환한다.
- `pending → active` 전환에서 형상 유효성 검사에 실패하면 400 오류를 반환한다. 패턴 상태는 변경되지 않는다.
- `decided_at` 은 서버가 자동으로 기록하며 요청 본문으로 지정할 수 없다.

## provenance

D27 에서 `pending → active` 전환 시점에 형상 유효성 검사를 수행하도록 확정되었다. 종료 상태의 불가역성과 `decided_at` 자동 기록도 같은 결정에서 비롯되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/pattern.rs` 에 위치한다.

## 미확정 (OPEN)

- `direct` 유형 패턴의 형상 유효성 검사 조건이 명시되지 않았다. 별도 필수 필드가 없는 유형인지 확인이 필요하다.
- `pattern_lifecycle` SSE 이벤트의 페이로드 구조(이전 상태 포함 여부 등)가 정의되지 않았다.
- `dissensus` 종료 이후 새 패턴을 자동으로 생성하거나 재시작하는 흐름이 이 엔드포인트 외부에서 별도로 처리되는지 확인이 필요하다.
