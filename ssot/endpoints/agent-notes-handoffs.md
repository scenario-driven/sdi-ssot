---
id: endpoint.agent-notes-handoffs
kind: Endpoint
title: "GET /agent_notes/handoffs"
definition: >
  특정 에이전트에게 전달된 미처리(pending) 인계 노트 전체를 반환하는 엔드포인트.
  에이전트가 세션을 시작할 때 이전 에이전트로부터 인계받은 작업이 있는지 확인하기 위해 호출한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/agent_note.rs"
relatesTo:
  - to: concept.handoff
    type: reads
  - to: concept.agent-note
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

에이전트가 새 세션을 시작하는 시점에 이전 에이전트가 자신에게 남겨둔 인계 노트가 있는지 확인하기 위해 호출하는 엔드포인트다. `to_agent` 필드가 자신의 식별자로 설정된 노트 중 아직 수신 확인(`receipt_acknowledged_at`)이 기록되지 않은 노트를 모두 반환한다. 읽기 전용이며 노트를 변경하지 않는다.

## 요청 / 응답

**GET /agent_notes/handoffs**

쿼리 파라미터:

- `to_agent` (필수): 인계 노트를 받을 대상 에이전트의 식별자.

응답: 해당 에이전트에게 전달된 미확인 인계 노트 목록. 각 노트에는 발신 에이전트(`from_agent`), 작성 시각, 본문, 첨부 페이로드 등이 포함된다.

## 권한 / 제약

- 읽기 전용. 노트를 변경하지 않으며 SSE 이벤트를 발행하지 않는다.
- 인계 노트를 읽은 후에는 반드시 `/agent_notes/:id/ack`를 호출해 수신 확인을 기록해야 한다. 확인 없이 방치하면 다음 세션에서도 동일한 노트가 반복 노출된다.
- 퇴역된 노트는 이 목록에 포함되지 않는다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/agent_note.rs`에서 유추됨. 에이전트 세션 시작 시 인계 확인 패턴(M2)에서 참조.

## 미확정 (OPEN)

- "미처리"의 정확한 정의가 `receipt_acknowledged_at IS NULL`인지, 별도 상태 컬럼이 있는지 미확인.
- 퇴역된 인계 노트도 별도 파라미터로 조회할 수 있는지 미확인.
- 결과의 정렬 기준(생성 시각 오름차순/내림차순 등) 미확인.
