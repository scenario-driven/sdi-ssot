---
id: endpoint.agent-notes-retire
kind: Endpoint
title: "POST /agent_notes/:id/retire"
definition: >
  에이전트 노트를 소프트 퇴역 처리하는 엔드포인트.
  오래되거나 더 이상 유효하지 않은 노트를 이력을 보존한 채 활성 목록에서 제외할 때 사용한다.
  노트 행 자체는 삭제되지 않으며, retired_at 타임스탬프와 이유가 기록된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/agent_note.rs"
relatesTo:
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

더 이상 활성 상태로 유지할 필요가 없는 에이전트 노트를 정리하기 위한 소프트 삭제 엔드포인트다. 이 엔드포인트를 호출하면 해당 노트에 `retired_at` 타임스탬프와 `retired_reason`이 기록된다. 노트 행 자체는 데이터베이스에 남아 이력 조회가 가능하지만, 활성 노트를 반환하는 `GET /agent_notes`에서는 제외된다. 퇴역 사실은 `agent_note.retired` SSE 이벤트로 발행된다.

## 요청 / 응답

**POST /agent_notes/:id/retire**

경로 파라미터:

- `id`: 퇴역 처리할 에이전트 노트의 식별자.

요청 본문:

- `reason` (필수, 비어 있으면 안 됨): 이 노트를 퇴역 처리하는 이유. 감사 추적과 맥락 이해를 위해 반드시 기술해야 한다.

응답: 퇴역 처리된 노트 행. `retired_at`과 `retired_reason` 필드가 채워진 상태로 반환된다.

## 권한 / 제약

- `reason` 필드는 비어 있을 수 없다.
- 이미 퇴역된 노트에 재호출 시의 동작(멱등 처리 또는 오류) 명세에서 미확인.
- 노트는 추가 전용 원칙에 따라 물리적으로 삭제되지 않는다. 퇴역은 소프트 삭제다.
- `agent_note.retired` SSE 이벤트가 발행된다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/agent_note.rs`에서 유추됨. 추가 전용 블랙보드 정책과 소프트 퇴역 패턴에서 참조.

## 미확정 (OPEN)

- 이미 퇴역된 노트에 재호출 시 멱등 처리하는지 오류를 반환하는지 미확인.
- 퇴역된 노트를 별도 파라미터로 조회(아카이브 조회)할 수 있는 엔드포인트가 있는지 미확인.
- 퇴역 처리를 되돌리는(un-retire) 기능이 있는지 미확인.
- 인계 노트(to_agent가 있는 노트)를 수신 확인 없이 퇴역 처리했을 때의 처리 방침 미확인.
