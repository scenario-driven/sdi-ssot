---
id: endpoint.comments
kind: Endpoint
title: "POST /comments, GET /comments"
definition: 특정 엔티티에 댓글을 남기거나 엔티티에 달린 댓글 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
relatesTo:
  - to: concept.plan
    type: reads
    note: "plan_id 를 앵커로 사용할 수 있다"
  - to: concept.task
    type: reads
    note: "task_id 를 앵커로 사용할 수 있다"
  - to: concept.scenario
    type: reads
    note: "scenario_id 를 앵커로 사용할 수 있다"
  - to: concept.round
    type: reads
    note: "round_id 를 앵커로 사용할 수 있다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

댓글은 플랜, 태스크, 시나리오, 라운드, 또는 프로젝트 수준의 엔티티에 자유 형식 텍스트를 남기는 협업 도구다. 에이전트와 사람 모두 댓글을 남길 수 있으며, `author` 기본값은 `"agent"` 다.

## 요청 / 응답

**POST /comments — 댓글 생성**

요청 본문:

- `body` — 댓글 내용 텍스트. 필수.
- `author` — 댓글 작성자. 선택. 기본값은 `"agent"`.
- 앵커 식별자 중 정확히 하나만 포함해야 한다.
  - `plan_id` — 플랜에 달리는 댓글.
  - `task_id` — 태스크에 달리는 댓글.
  - `scenario_id` — 시나리오에 달리는 댓글.
  - `round_id` — 라운드에 달리는 댓글.
  - `project_id` — 프로젝트 수준 댓글.

앵커 식별자를 두 개 이상 보내면 400 오류를 반환한다. 앵커 식별자를 하나도 포함하지 않아도 400 오류를 반환한다.

성공 응답: 생성된 댓글 객체. `comment.created` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

**GET /comments — 댓글 목록 조회**

쿼리 파라미터로 앵커 식별자 중 하나를 반드시 지정해야 한다(`plan_id`, `task_id`, `scenario_id`, `round_id`, `project_id` 중 하나). 지정된 앵커에 달린 댓글 배열을 시간순으로 반환한다.

## 권한 / 제약

- `POST` 에서 앵커가 하나도 없거나 둘 이상이면 400 오류.
- `GET` 에서 앵커 쿼리 파라미터 없이 호출하면 400 오류.
- 앵커로 지정한 엔티티가 존재하지 않으면 404 오류.

## provenance

에이전트와 사람이 플랜·태스크·시나리오·라운드 전반에 걸쳐 협업 맥락을 공유할 수 있도록 댓글 기능이 설계되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/collab.rs` 에 위치한다.

## 미확정 (OPEN)

- `comment.created` SSE 이벤트의 채널이 앵커 엔티티 단위인지 전역 단위인지 확인이 필요하다.
- 댓글에 대한 댓글(스레드)을 지원하는지, 평면 목록만 허용하는지 명시되지 않았다.
- `GET` 응답의 정렬 기준(생성 시각 오름차순/내림차순)이 명시되지 않았다.
- 페이지네이션 지원 여부가 확인되지 않았다.
