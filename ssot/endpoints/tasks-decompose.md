---
id: endpoint.tasks-decompose
kind: Endpoint
title: "POST /tasks/:id/decompose"
definition: 하나의 부모 태스크를 여러 개의 서브태스크로 분해한다. 각 서브태스크는 부모 태스크와 연결되며, 부모의 협업 패턴 식별자를 상속한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
relatesTo:
  - to: concept.task
    type: mutates
  - to: concept.scenario
    type: reads
    note: "parent_scenario_ids가 각 서브태스크에 전파된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

부모 태스크(`:id`)를 여러 개의 서브태스크로 분해(decompose)하는 엔드포인트다. LLM 에이전트가 복잡한 태스크를 작은 단위로 쪼개어 병렬 또는 순차 실행을 준비할 때 사용한다.

`POST /tasks/:id/subtasks`라는 별칭 경로로도 접근 가능하다.

분해 시 각 서브태스크는 부모 태스크와 parent 관계로 연결된다. 부모 태스크가 어떤 협업 패턴(`produced_via_pattern_id`)으로 만들어졌는지 정보가 자동으로 각 서브태스크에 상속된다(D23). 빈 서브태스크 목록은 요청이 거부된다.

## 요청 / 응답

**경로 파라미터**

- `:id` — 분해할 부모 태스크의 식별자.

**요청 본문 (JSON)**

- `round_id` (선택) — 서브태스크를 배정할 라운드 식별자. 생략하면 부모 태스크가 속한 라운드를 기본값으로 사용한다.
- `subtasks` (필수, 배열) — 생성할 서브태스크 목록. 빈 배열이면 400으로 거부된다. 각 항목은 다음 필드를 포함한다:
  - `short_code` (필수) — 서브태스크를 식별하는 짧은 코드.
  - `description` (필수) — 서브태스크가 해야 할 일에 대한 설명.
  - `parent_scenario_ids` (선택) — 이 서브태스크가 구현 또는 검증해야 할 시나리오 식별자 목록.
  - `parent_requirement_ids` (선택) — 이 서브태스크가 충족해야 할 요구사항 식별자 목록.

**응답**

생성된 서브태스크 전체 목록과 갱신된 부모 태스크 정보를 반환한다.

**SSE 이벤트**

- `task.created` — 서브태스크 하나가 생성될 때마다 발행된다. 서브태스크 수만큼 발행된다.
- `task.decomposed` — 부모 태스크의 분해가 완료된 시점에 한 번 발행된다.

## 권한 / 제약

- `subtasks` 배열이 비어있으면 400을 반환한다.
- 부모 태스크가 존재하지 않으면 404를 반환한다.
- 각 서브태스크에는 부모의 `produced_via_pattern_id`가 자동으로 설정된다. 서브태스크별로 개별 패턴을 지정하는 기능이 지원되는지는 미확인이다.
- `round_id`를 명시하지 않으면 부모 태스크의 `round_id`가 상속된다. 부모 태스크에도 `round_id`가 없을 경우의 동작은 미확인이다.

## provenance

D23 정책(서브태스크의 패턴 상속)과 연관된 엔드포인트다. 복잡한 태스크를 LLM이 자율적으로 분해하여 병렬 실행하는 시나리오를 지원하기 위해 설계되었다. `POST /tasks/:id/subtasks` 별칭은 의미론적 명확성을 위한 동의어 경로다. `sdi-plugin/crates/daemon/src/router/run.rs`에 구현되어 있다고 추정된다.

## 미확정 (OPEN)

- 이미 서브태스크가 존재하는 부모 태스크에 재호출할 경우 기존 서브태스크를 유지하고 추가하는지, 오류를 반환하는지 명시되지 않았다.
- 서브태스크별로 개별 `produced_via_pattern_id`를 오버라이드할 수 있는지 미확인이다.
- `parent_scenario_ids`와 `parent_requirement_ids`가 비어있을 때 서브태스크 생성이 허용되는지 명시되지 않았다.
- `task.created` SSE의 발행 순서가 서브태스크 배열 순서를 보장하는지 미확인이다.
- `POST /tasks/:id/subtasks` 별칭 경로가 라우터에서 명시적으로 등록되어 있는지 코드 수준에서 미확인이다.
