---
id: endpoint.tasks-status
kind: Endpoint
title: "POST /tasks/:id/status"
definition: 특정 태스크의 상태를 변경한다. todo→in_progress, in_progress→blocked 등 도메인이 허용하는 전환만 적용되며, done 전환은 이 엔드포인트에서 명시적으로 거부된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
    type: mutates
    note: "태스크의 status 필드를 갱신한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

활성 태스크의 상태를 변경하는 엔드포인트다. 태스크 상태 머신이 허용하는 전환(todo→in_progress, in_progress→blocked, blocked→in_progress 등)만 적용된다. **done 전환은 이 엔드포인트에서 처리하지 않는다** — done은 증거(evidence)를 필수로 요구하며, 반드시 `/tasks/:id/complete`를 통해야 한다. 이 제약은 증거 없이 태스크가 완료로 표시되는 사고를 원천 차단하기 위한 도메인 규칙이다.

## 요청 / 응답

**요청**

- 경로 파라미터: `:id` — 변경할 태스크의 ULID
- 바디: 변경할 목표 상태(status) 문자열

**응답**

- 성공 시 변경 후의 태스크 상태를 반환한다.
- 허용되지 않는 전환(예: todo→blocked, 또는 done으로의 전환 시도)은 오류로 거부된다.
- 존재하지 않는 태스크 ID는 404로 응답한다.

## 권한 / 제약

- done 전환 시도는 이 엔드포인트에서 항상 거부된다. done 처리는 `/tasks/:id/complete`가 전담한다.
- 도메인이 정의하지 않은 상태값은 400으로 거부된다.
- 전환 가능 여부는 애플리케이션 계층이 아니라 도메인 계층에서 검증한다.

## provenance

SDI 플러그인 데몬의 태스크 라우터(`sdi-plugin/crates/daemon/src/router/task.rs`)에 구현되어 있다. done 전환의 분리는 태스크 완료와 증거 기록의 원자성을 보장하기 위한 설계 결정이다.

## 미확정 (OPEN)

- 상태 전환 목록(전체 허용 매트릭스)이 코드에 정의되어 있는지, 아니면 외부 설정으로 관리되는지 미확인.
- 상태 변경 시 SSE 이벤트를 발행하는지 여부 미확인.
- 동일 상태로의 전환(no-op)을 허용하는지 아니면 오류로 처리하는지 미확인.
