---
id: endpoint.usage
kind: Endpoint
title: "POST /usage, GET /usage"
definition: LLM 토큰 사용량과 비용을 기록하고 조회하는 엔드포인트. POST 로 사용 기록을 추가하면 SSE 이벤트가 발행된다. GET 으로 프로젝트·계획·태스크 기준으로 사용 기록을 조회한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.usage-budget
    type: mutates
    note: ""
  - to: concept.run
    type: reads
    note: ""
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

LLM 호출로 발생한 토큰 사용량과 비용을 기록하고 조회하는 엔드포인트다. POST 로 새로운 사용 기록을 추가하면 `usage.recorded` SSE 이벤트가 발행된다. GET 으로는 프로젝트, 계획, 또는 태스크 기준으로 기록된 사용량 목록을 조회할 수 있다.

## 요청 / 응답

**POST /usage** — 사용량 기록 추가
- 요청 본문:
  - `project_id`(필수): 어느 프로젝트의 사용량인지
  - `plan_id`(선택): 연관 계획 ID
  - `task_id`(선택): 연관 태스크 ID
  - `run_id`(선택): 연관 실행 ID
  - `model`(필수): 사용한 모델 이름
  - `tier`(선택): 모델 티어 구분
  - `input_tokens`(필수): 입력 토큰 수
  - `output_tokens`(필수): 출력 토큰 수
  - `cache_read`(필수): 캐시에서 읽은 토큰 수
  - `cache_write`(필수): 캐시에 쓴 토큰 수
  - `tool_calls`(필수): 도구 호출 횟수
  - `cost_usd`(필수): 해당 호출의 비용(USD)
- 응답: 생성된 사용 기록 객체
- 사이드이펙트: `usage.recorded` SSE 이벤트 발행

**GET /usage** — 사용량 기록 목록 조회
- 쿼리 파라미터: `project_id`, `plan_id`, `task_id` 중 하나 이상 필수
- 응답: 사용 기록 배열

## 권한 / 제약

- GET 호출 시 `project_id`, `plan_id`, `task_id` 중 하나도 지정하지 않으면 400 오류를 반환한다.
- `cost_usd` 는 호출자가 직접 계산하여 제출하며, 서버가 자동으로 계산하지 않는다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보, 요청 필드 구조, SSE 이벤트 이름은 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `cache_read`, `cache_write`, `tool_calls` 가 선택 필드인지 필수 필드인지 명세에서 확정되지 않음.
- GET 시 복수의 필터(예: `plan_id` 와 `task_id` 동시 지정) 를 조합할 수 있는지 미확인.
- `tier` 필드의 허용 값 목록(예: "lite", "standard", "pro") 이 확정되지 않음.
