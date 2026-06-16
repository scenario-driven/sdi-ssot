---
id: endpoint.agent-notes-ack
kind: Endpoint
title: "POST /agent_notes/:id/ack"
definition: >
  인계 노트의 수신을 확인하는 엔드포인트. 수신 에이전트가 인계 내용을 읽은 후 호출하여 수신 확인 시각을 기록한다.
  이 호출 이후 해당 노트는 미처리 인계 목록에서 제외된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/agent_note.rs"
relatesTo:
  - to: concept.handoff
    type: mutates
  - to: concept.agent-note
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

인계 노트를 받은 에이전트가 해당 노트를 읽었음을 시스템에 알리는 엔드포인트다. 호출하면 해당 노트에 `receipt_acknowledged_at` 타임스탬프가 기록된다. 이후 `GET /agent_notes/handoffs` 조회 시 이 노트는 더 이상 미처리 목록에 포함되지 않는다. 확인 사실은 `agent_note.acknowledged` SSE 이벤트로 발행된다.

## 요청 / 응답

**POST /agent_notes/:id/ack**

경로 파라미터:

- `id`: 수신 확인할 에이전트 노트의 식별자.

요청 본문: 없음 (또는 빈 객체).

응답: 수신 확인 처리된 노트 행. `receipt_acknowledged_at` 필드가 채워진 상태로 반환된다.

## 권한 / 제약

- 이미 수신 확인된 노트에 다시 ack를 보내는 경우의 동작(멱등 처리 여부 또는 오류 반환)은 명세에서 확인되지 않았다.
- 수신 확인은 인계 노트(`to_agent`가 설정된 노트)에만 의미가 있다. 일반 관찰 노트에 호출했을 때의 동작은 미확인.
- `agent_note.acknowledged` SSE 이벤트가 발행되어 발신 에이전트나 감시 컴포넌트가 인계 완료를 감지할 수 있다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/agent_note.rs`에서 유추됨. M2(인계 영수증) 패턴에서 ack 단계로 참조.

## 미확정 (OPEN)

- 이미 ack된 노트에 재호출 시 멱등 처리하는지, 오류를 반환하는지 미확인.
- `to_agent`가 없는 일반 노트에 ack 호출 시 허용 여부 미확인.
- SSE 이벤트 페이로드에 발신·수신 에이전트 식별자가 포함되는지 미확인.
