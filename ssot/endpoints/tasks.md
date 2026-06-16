---
id: endpoint.tasks
kind: Endpoint
title: "POST /tasks, GET /tasks"
definition: 태스크를 생성하거나 특정 라운드에 속한 태스크 목록을 조회하는 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
relatesTo:
  - to: concept.task
    type: mutates
  - to: concept.round
    type: reads
  - to: concept.scenario
    type: reads
    note: "parent_scenario_ids를 통해 태스크가 구현 대상 시나리오와 연결됨"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

태스크는 시나리오와 요구사항을 LLM이 분해하여 만들어내는 실행 단위다. 사람이 직접 태스크를 작성하지 않는다(D3). POST는 새 태스크를 만들고, GET은 특정 라운드에 속한 태스크 전체를 나열한다.

패턴이 지정되지 않으면 D23 결정에 따라 직접 센티넬(direct sentinel)이 적용된다.

태스크가 생성되면 `task.created` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /tasks**

요청 본문(JSON):

- `round_id` (필수): 이 태스크가 속할 라운드 식별자.
- `short_code` (필수): 태스크를 식별하는 짧은 코드.
- `description` (필수): 태스크가 수행해야 하는 내용을 자연어로 서술한 설명.
- `parent_scenario_ids` (선택): 이 태스크가 구현 대상으로 삼는 시나리오 ID 목록.
- `parent_requirement_ids` (선택): 이 태스크가 참조하는 요구사항 ID 목록.
- `produced_via_pattern_id` (선택): 이 태스크를 생성한 협업 패턴 ID. 없으면 직접 센티넬 적용(D23).

응답: 생성된 태스크 객체. 상태는 초기값(`todo`) 포함.

**GET /tasks**

쿼리 파라미터:

- `round_id` (필수): 조회할 라운드의 식별자.

응답: 해당 라운드에 속한 태스크 배열. 각 항목에 상태, 설명, 연결된 시나리오·요구사항 ID 포함.

## 권한 / 제약

- GET 시 `round_id` 쿼리 파라미터가 없으면 요청이 거부된다.
- 태스크는 LLM 에이전트가 생성하는 것이 원칙이며, 사람이 직접 작성하는 것은 설계 의도에 반한다(D3).
- 패턴이 없으면 직접 센티넬로 대체된다(D23).

## provenance

D3(LLM이 시나리오+요구사항으로부터 태스크 분해), D23(패턴 없을 때 직접 센티넬 폴백) 결정에 근거한다. `parent_scenario_ids` 연결은 이 태스크가 어떤 시나리오를 구현하기 위한 것인지 추적 가능하게 하는 핵심 연결고리다.

## 미확정 (OPEN)

- 태스크 초기 상태가 `todo`인지 다른 값인지 미확인.
- `task.created` SSE 이벤트의 페이로드 스키마 세부 사항 미확인.
- GET 응답에 페이지네이션이 적용되는지 미확인.
- `parent_scenario_ids`가 비어 있을 때(순수 요구사항 기반 태스크) 허용되는지 미확인.
