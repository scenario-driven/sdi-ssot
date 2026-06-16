---
id: endpoint.decisions
kind: Endpoint
title: "POST /decisions, GET /decisions"
definition: 결정(decision)을 생성하거나 플랜별로 목록을 조회한다. 생성 시 M3 협상 단계를 분류하는 kind 필드와 번복 계획, 영향 범위 점수 등 다양한 메타데이터를 함께 기록할 수 있다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
relatesTo:
  - to: concept.decision
    type: mutates
    note: "새 결정 레코드를 생성하거나 기존 목록을 조회한다."
  - to: concept.consensus
    type: mutates
    note: "D20: kind=consensus 또는 kind=dissensus일 때 특정 이벤트가 트리거된다."
  - to: concept.plan
    type: reads
    note: "결정은 반드시 특정 플랜에 귀속된다. GET 조회 시 plan_id가 필수다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

이 엔드포인트는 POST(생성)와 GET(목록 조회) 두 메서드를 함께 다룬다.

**POST — 결정 생성**

새로운 결정을 기록한다. 결정은 플랜 내에서 내려진 설계·아키텍처·프로세스상의 선택을 영구적으로 남기는 기록 단위다.

`kind` 필드는 D20 규칙에 따라 M3 4단계 협상의 어느 단계에 해당하는 결정인지를 분류한다:
- `proposal`: 초기 제안 단계
- `critique`: 비판·검토 단계
- `consensus`: 합의 도달
- `dissensus`: 이견 발생 — 이 종류로 생성된 결정은 자동으로 `escalated_at` 타임스탬프가 기록된다.

D28 번복 지원 필드도 함께 제공된다:
- `reversal_plan`: 이 결정을 번복하는 절차를 기술한 JSON
- `blast_radius_score`: 결정의 영향 범위를 0–10 점수로 표현 (기본값 5)
- `reversal_of`: 이 결정이 특정 결정의 롤백임을 나타내는 참조 ID

D23 규칙에 따라 패턴이 없는 경우 직접 센티넬(sentinel)로 해소된다.

`status` 필드의 기본값은 `accepted`이며, `kind`의 기본값은 `proposal`이다.

**GET — 결정 목록 조회**

특정 플랜에 속한 결정 목록을 반환한다. `plan_id`는 필수 쿼리 파라미터다.

## 요청 / 응답

**POST 요청 바디**

| 필드 | 필수 여부 | 기본값 | 설명 |
|---|---|---|---|
| plan_id | 필수 | — | 귀속될 플랜의 ULID |
| short_code | 필수 | — | 사람이 읽는 식별자 (예: D-001) |
| title | 필수 | — | 결정의 제목 |
| body | 필수 | — | 결정의 상세 내용 |
| status | 선택 | accepted | 초기 상태 |
| kind | 선택 | proposal | M3 협상 단계 분류 |
| reversal_plan | 선택 | — | 번복 절차 (JSON) |
| blast_radius_score | 선택 | 5 | 영향 범위 점수 (0–10) |
| reversal_of | 선택 | — | 롤백 대상 결정의 ID |

**GET 요청**

- 쿼리 파라미터: `plan_id` (필수)

**응답**

- POST: 생성된 결정 레코드를 반환한다. `kind=dissensus`인 경우 `escalated_at`이 채워진다.
- GET: 해당 플랜의 결정 목록을 반환한다.

## 권한 / 제약

- GET에서 `plan_id` 없이 조회하면 거부된다.
- `kind=dissensus`는 생성 즉시 `escalated_at`을 자동 기록한다 — 별도 API 호출 불필요.
- `blast_radius_score`는 0 이상 10 이하의 정수여야 한다.
- `reversal_of`로 참조된 결정 ID가 실제로 존재해야 하는지 여부는 미확인.

## provenance

SDI 플러그인 데몬의 결정 라우터(`sdi-plugin/crates/daemon/src/router/decision.rs`)에 구현되어 있다. D20(M3 4단계 협상), D23(직접 센티넬), D28(번복 계획) 설계 결정이 이 엔드포인트의 스키마를 직접 형성했다.

## 미확정 (OPEN)

- `kind=consensus` 생성 시 트리거되는 구체적인 이벤트(SSE 발행 여부, 플랜 상태 자동 전환 등) 미확인.
- `agent_name` 필드가 POST 바디에서 받는지 아니면 인증 컨텍스트에서 주입되는지 미확인.
- GET 응답의 정렬 기준 및 페이지네이션 지원 여부 미확인.
- `status` 초기값을 `accepted`가 아닌 다른 값으로 생성하는 유효한 케이스가 있는지 미확인.
