---
id: endpoint.decision
kind: Endpoint
title: "GET /decisions/:id"
definition: 단일 결정(decision)을 ID로 조회한다. 상태, 종류, 협상 체인, 번복 계획, 영향 범위 점수 등 해당 결정의 모든 필드를 포함하여 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
relatesTo:
  - to: concept.decision
    type: reads
    note: "단일 결정 레코드의 모든 필드를 반환한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

결정의 ULID를 경로 파라미터로 받아 해당 결정의 전체 상세 정보를 반환하는 조회 전용 엔드포인트다. 목록 조회(`GET /decisions`)가 요약 정보를 제공하는 반면, 이 엔드포인트는 특정 결정의 모든 메타데이터를 포함한 완전한 레코드를 반환한다.

반환되는 필드에는 다음이 포함된다:
- `status`: 결정의 현재 상태 (accepted, rejected, superseded 등)
- `kind`: M3 협상 단계 분류 (proposal, critique, consensus, dissensus)
- `proposal_id` 체인: 이 결정이 협상 흐름에서 어느 제안으로부터 파생되었는지 추적하는 연결 정보
- `agent_name`: 이 결정을 기록한 에이전트 이름
- `escalated_at`: dissensus 결정에서 자동으로 기록되는 에스컬레이션 타임스탬프
- `reversal_plan`: 번복 절차를 기술한 JSON
- `blast_radius_score`: 영향 범위 점수 (0–10)
- `reversal_of`: 이 결정이 롤백하는 대상 결정의 ID
- `supersede_when`: 이 결정이 다른 결정을 언제 대체하는지를 정의하는 조건

## 요청 / 응답

**요청**

- 경로 파라미터: `:id` — 조회할 결정의 ULID

**응답**

- 해당 결정의 모든 필드를 포함한 레코드를 반환한다.
- 존재하지 않는 ID는 404로 응답한다.

## 권한 / 제약

- 쓰기 작업이 없는 순수 조회 엔드포인트다.
- 결정은 플랜에 귀속되지만, 이 엔드포인트는 플랜 ID 없이 결정 ID만으로 직접 접근 가능하다.

## provenance

SDI 플러그인 데몬의 결정 라우터(`sdi-plugin/crates/daemon/src/router/decision.rs`)에 구현되어 있다. `endpoint.decisions`(목록/생성)와 `endpoint.decisions-status`(상태 변경)와 함께 결정 도메인의 CRUD 인터페이스를 구성한다.

## 미확정 (OPEN)

- `proposal_id` 체인이 단일 참조인지 아니면 협상 흐름 전체를 재구성할 수 있는 연결 리스트인지 미확인.
- `supersede_when` 필드의 값 형식(조건식, 타임스탬프, 이벤트명 등) 미확인.
- `agent_name`이 자유 텍스트인지 아니면 등록된 에이전트 식별자를 참조하는지 미확인.
- 소프트 삭제(soft delete)된 결정을 이 엔드포인트로 조회할 수 있는지 미확인.
