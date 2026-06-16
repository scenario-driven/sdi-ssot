---
id: endpoint.scenarios
kind: Endpoint
title: "POST /scenarios, GET /scenarios"
definition: GWT 시나리오를 생성하거나, 특정 플랜에 속한 시나리오 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario
    type: mutates
  - to: concept.given-when-then
    type: mutates
  - to: concept.scenario-claim
    type: mutates
    note: "D29 claimed_resources_json이 생성 시점에 함께 저장된다"
  - to: concept.plan
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

GWT(Given/When/Then) 시나리오 컬렉션 엔드포인트다. POST로 새 시나리오를 만들고, GET으로 플랜에 속한 시나리오 전체를 나열한다. 시나리오는 SDI에서 구현·검증의 1등 시민이며, 이 엔드포인트가 그 시나리오를 최초로 등록하는 진입점이다. M4 계약(에이전트 이름 명시), D5 규칙(GWT 비어있음 금지), D23(패턴 출처 자동 해소), D29(자원 선점 경로 글로브) 등 여러 설계 결정이 이 엔드포인트에서 교차한다.

## 요청 / 응답

**POST /scenarios**
- 요청 본문(JSON):
  - `plan_id`(필수): 시나리오를 귀속시킬 플랜 ID.
  - `short_code`(필수): 시나리오 단축 식별 코드(예: SC-01).
  - `given`(필수): 사전 조건 서술. 비어있으면 D5 규칙에 의해 거부한다.
  - `when`(필수): 동작/이벤트 서술. 비어있으면 D5 규칙에 의해 거부한다.
  - `then`(필수): 기대 결과 서술. 비어있으면 D5 규칙에 의해 거부한다.
  - `tags`(선택): 분류 태그 배열.
  - `confirmed`(선택): true로 지정하면 초안 상태를 거치지 않고 즉시 확정(confirmed) 상태로 생성된다.
  - `depends_on`(선택): 다른 시나리오의 short_code 목록. DAG 순서 의존성을 표현한다.
  - `produced_by`(선택): 이 시나리오를 생성한 에이전트 이름(M4 계약).
  - `verified_by`(선택): 이 시나리오를 검증할 에이전트 이름(M4 계약).
  - `claimed_resources_json`(선택): D29 규칙에 따라 이 시나리오가 독점적으로 사용할 파일 경로 글로브 목록(JSON 형식). 생성 시점에 시나리오 선점(claim) 레코드로 함께 저장된다.
  - `produced_via_pattern_id`(선택): D23 규칙 — 이 시나리오를 생성한 협업 패턴 ID. 지정하지 않으면 활성 패턴이 없는 경우 서버가 직접(direct) 패턴 출처로 자동 해소한다.
- D5 검증: `given`, `when`, `then` 셋 모두 비어있지 않아야 한다. 하나라도 비어있으면 거부한다.
- 응답: 생성된 시나리오 객체.

**GET /scenarios**
- 쿼리 파라미터 `plan_id`(필수): 조회할 플랜 ID. 없으면 요청을 거부한다.
- 응답: 해당 플랜에 속한 시나리오 목록(배열).

## 권한 / 제약

- **D5**: `given`, `when`, `then` 세 필드 모두 비어있으면 안 된다. 공백만 있는 경우도 비어있는 것으로 처리한다고 가정하나, 구체 판단 기준은 미확인.
- `confirmed=true`로 생성하면 초안(draft) 확정(confirm) 흐름을 건너뛰므로 신중히 사용해야 한다.
- `depends_on`에 기재된 short_code들은 같은 플랜 내에 존재해야 하며, 순환 의존은 허용하지 않는다(추정).
- GET 시 `plan_id` 쿼리 파라미터는 필수다. 전체 조회는 허용하지 않는다.

## provenance

sdi-plugin 시나리오 라우터(`scenario.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. D5·D23·D29·M4 계약 관련 동작은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- `depends_on` 순환 의존 감지 및 거부 여부 미확인.
- `claimed_resources_json`의 경로 글로브 형식 제약(어떤 glob 문법을 허용하는지) 미확인.
- `produced_by` / `verified_by` 에이전트 이름의 유효성 검증 여부 미확인.
- `confirmed=true` 생성 시 D8 게이트(플랜 승인 전 생성 가능 여부)와의 관계 미확인.
- GET 결과의 정렬 기준(생성 순서, short_code 순서, depends_on 위상 정렬 등) 미확인.
- `tags` 필드의 값 형식 및 허용 최대 개수 미확인.
