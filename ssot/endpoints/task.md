---
id: endpoint.task
kind: Endpoint
title: "GET /tasks/:id"
definition: 특정 태스크 하나의 상세 정보를 조회하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
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

태스크 ID를 경로 파라미터로 받아 해당 태스크의 전체 정보를 돌려준다. 현재 상태, 첨부된 증거, 연결된 시나리오·요구사항 ID, 그리고 이 태스크를 만들어낸 협업 패턴 정보를 포함한다.

## 요청 / 응답

**GET /tasks/:id**

경로 파라미터:

- `id` (필수): 조회할 태스크의 식별자.

응답(JSON):

- `id`: 태스크 식별자.
- `round_id`: 이 태스크가 속한 라운드 식별자.
- `short_code`: 태스크의 짧은 코드.
- `description`: 태스크 설명.
- `status`: 현재 상태 (`todo`, `in_progress`, `done`, `cancelled`, `blocked` 등).
- `evidence`: 이 태스크와 연결된 증거 데이터. 테스트 실행 결과, 파일 변경 목록 등이 포함될 수 있다.
- `parent_scenario_ids`: 이 태스크가 구현 대상으로 삼는 시나리오 ID 목록.
- `parent_requirement_ids`: 이 태스크가 참조하는 요구사항 ID 목록.
- `produced_via_pattern_id`: 이 태스크를 생성한 협업 패턴 ID. 직접 센티넬로 생성된 경우 null.
- `created_at`: 생성 시각.
- `updated_at`: 마지막 수정 시각.

## 권한 / 제약

- 존재하지 않는 ID를 조회하면 404를 반환한다.
- 읽기 전용 엔드포인트이며 상태를 변경하지 않는다.

## provenance

태스크의 `produced_via_pattern_id`는 이 태스크가 어떤 협업 패턴 흐름에서 도출되었는지 추적하는 연결고리다. D23에 따라 패턴 없이 직접 생성된 경우에는 null이 된다. `evidence` 필드는 검증 단계에서 테스트 러너가 기록한 결과물이 누적되는 공간이다.

## 미확정 (OPEN)

- `evidence` 필드의 정확한 데이터 구조(단순 문자열 배열인지, 구조화된 객체인지) 미확인.
- `status` 값의 전체 열거 목록 미확인 (`blocked` 외에 추가 상태가 있는지).
- `parent_scenario_ids`와 `parent_requirement_ids`가 빈 배열인지 null인지(태스크에 연결된 시나리오·요구사항이 없을 때) 미확인.
