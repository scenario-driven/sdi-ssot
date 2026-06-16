---
id: endpoint.decisions-rollback
kind: Endpoint
title: "POST /decisions/:id/rollback"
definition: 특정 결정을 취소하는 새 합의 결정을 추가한다. 원본 결정은 절대 수정되지 않으며, 취소 기록은 별도의 새 행으로 append된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
relatesTo:
  - to: concept.decision
    type: mutates
  - to: concept.consensus
    type: mutates
    note: "rollback 행은 reversal_of 필드가 원본 결정 id를 가리키는 consensus 종류의 결정이다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

특정 결정(`:id`)을 취소(rollback)할 때 사용하는 엔드포인트다. 이 엔드포인트를 호출하면 원본 결정을 직접 수정하거나 삭제하지 않고, 원본을 가리키는 `reversal_of` 필드를 포함한 새로운 합의(consensus) 결정 행이 데이터베이스에 추가된다. 시스템은 기록을 항상 추가(append-only) 방식으로 유지한다는 원칙(D12)을 따르기 때문에, 과거 결정은 어떤 경우에도 덮어쓰거나 삭제되지 않는다.

원본 결정의 상태를 "무효화됨" 또는 "대체됨"으로 표시하려면 이 엔드포인트 호출 이후 별도로 `POST /decisions/:id/status`를 호출해야 한다. 이 엔드포인트 자체는 원본 상태를 변경하지 않는다.

## 요청 / 응답

**요청 경로 파라미터**

- `:id` — 취소하려는 원본 결정의 식별자.

**요청 본문 (JSON)**

- `short_code` (필수) — 취소 결정을 식별하는 짧은 코드. 예: `ROLLBACK-DEC-007`.
- `title` (필수) — 취소 결정의 제목. 무엇을 왜 되돌리는지 한 줄로 설명한다.
- `body` (필수) — 취소 배경과 영향에 대한 상세 설명.
- `reversal_plan` (필수, JSON) — 실제 취소 실행 계획을 담은 구조화된 JSON. 비어있거나 형식이 잘못된 경우 요청이 거부된다(400).
- `blast_radius_score` (선택, 0–10 범위 정수, 기본값 5) — 이 취소 작업이 시스템에 미치는 영향 범위 추정치.
- `produced_via_pattern_id` (선택) — 이 취소 결정을 생성한 협업 패턴 식별자.
- `agent_name` (선택, 기본값 `"reversal-runner"`) — 취소를 수행하는 에이전트 이름.

**응답**

성공 시 새로 생성된 취소 결정 행의 데이터를 반환한다. `reversal_of` 필드에 원본 결정 id가 기록되어 있다.

**SSE 이벤트**

- `rollback_initiated` — 취소 프로세스 시작 시 발행.
- `rollback_completed` — 새 취소 결정 행이 성공적으로 저장된 후 발행.

## 권한 / 제약

- `reversal_plan` JSON은 삽입 전 유효성 검사를 통과해야 한다. 유효하지 않으면 요청 전체가 거부된다.
- 원본 결정 행은 어떤 경우에도 수정되지 않는다(D12 append-only 원칙).
- `blast_radius_score`는 0 이상 10 이하의 정수여야 한다.
- 원본 결정이 존재하지 않으면 404를 반환한다.

## provenance

D28 결정(Rollback Plan 정책)에서 도출되었다. 취소 행을 별도 합의 결정으로 append하는 방식은 D12 append-only 불변 원칙과의 충돌을 방지하기 위한 설계다. `sdi-plugin/crates/daemon/src/router/decision.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- `reversal_plan` JSON의 구체적인 스키마(필수 키 목록, 중첩 구조)가 코드 수준에서 확인되지 않았다.
- `rollback_initiated`와 `rollback_completed` SSE 이벤트의 페이로드 구조가 미확인 상태다.
- 이미 취소된 결정(`reversal_of`가 이미 존재하는 경우)을 다시 취소하려 할 때의 동작이 명시되지 않았다.
- 원본 결정의 상태 변경(superseded)을 이 엔드포인트가 자동으로 수행하는지, 호출자가 별도로 해야 하는지 코드 수준에서 미확인이다.
