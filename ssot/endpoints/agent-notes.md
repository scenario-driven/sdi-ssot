---
id: endpoint.agent-notes
kind: Endpoint
title: "POST /agent_notes, GET /agent_notes"
definition: >
  에이전트 노트를 추가하고 활성 노트 목록을 조회하는 엔드포인트.
  POST는 에이전트가 작업 중 관찰한 내용, 판단, 또는 다른 에이전트로의 인계 정보를 블랙보드에 기록한다.
  GET은 특정 범위에 연결된 활성(미퇴역) 노트를 반환한다.
  노트는 추가 전용(append-only)으로, 수정하거나 삭제할 수 없다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/agent_note.rs"
relatesTo:
  - to: concept.agent-note
    type: mutates
  - to: concept.handoff
    type: mutates
    note: "to_agent 필드가 지정되면 이 노트는 인계 노트(handoff note)가 된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

에이전트들이 공유하는 블랙보드 저널(M1)과 에이전트 간 인계 영수증(M2)을 구현하는 엔드포인트다. POST로 노트를 추가하면 영구히 보존되며, 어떤 에이전트도 이를 수정하거나 삭제할 수 없다(추가 전용). 다른 에이전트에게 작업을 넘길 때는 `to_agent` 필드를 채워 인계 노트로 만든다. GET은 퇴역(retired)되지 않은 활성 노트를 범위와 앵커 기준으로 필터링해 반환한다.

노트가 생성될 때마다 `agent_note.created` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /agent_notes**

요청 본문:

- `project_id` (선택): 노트가 속하는 프로젝트. 범위에 따라 생략 가능.
- `plan_id` (선택): 노트를 특정 플랜에 연결할 때 사용.
- `round_id` (선택): 노트를 특정 라운드에 연결할 때 사용.
- `scenario_id` (선택): 노트를 특정 시나리오에 연결할 때 사용.
- `task_id` (선택): 노트를 특정 태스크에 연결할 때 사용.
- `scope_kind` (필수): 노트의 범위 종류. 노트가 어떤 수준의 작업 단위에 속하는지 분류.
- `kind` (필수): 노트의 종류. 관찰, 판단, 인계, 알림 등으로 구분.
- `from_agent` (필수): 이 노트를 작성한 에이전트 식별자.
- `to_agent` (선택): 노트를 전달할 대상 에이전트. 이 필드가 채워지면 인계 노트가 된다.
- `body` (필수): 노트의 본문 텍스트.
- `payload` (선택): 구조화된 추가 데이터를 자유 형식으로 첨부.

응답: 생성된 노트 행.

**GET /agent_notes**

쿼리 파라미터:

- `scope_kind` (필수): 조회할 노트의 범위 종류.
- `anchor_id` (필수): 해당 범위에서 노트를 연결할 기준 앵커의 식별자. 예를 들어 `scope_kind`가 태스크면 `anchor_id`는 태스크 ID.
- `kind` (선택): 특정 종류의 노트만 필터링할 때 사용.

응답: 퇴역되지 않은 활성 노트 목록.

## 권한 / 제약

- 노트는 추가 전용이다. POST 이후 내용을 바꾸거나 행을 삭제하는 엔드포인트는 없다.
- 노트를 무효화하려면 `/agent_notes/:id/retire` 엔드포인트를 사용해 소프트 퇴역 처리한다.
- GET은 `scope_kind`와 `anchor_id` 두 파라미터가 모두 필수다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/agent_note.rs`에서 유추됨. M1(블랙보드 저널), M2(인계 영수증) 설계 패턴 참조.

## 미확정 (OPEN)

- `scope_kind`와 `kind`의 허용 열거값 전체 목록 미확인.
- `anchor_id`가 `scope_kind`에 따라 어떤 테이블의 어떤 컬럼을 참조하는지 구체적 매핑 미확인.
- `payload` 필드의 스키마 제약 또는 최대 크기 미확인.
- SSE 이벤트 페이로드 구조 미확인.
