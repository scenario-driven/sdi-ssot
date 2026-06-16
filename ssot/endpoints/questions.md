---
id: endpoint.questions
kind: Endpoint
title: "POST /questions, GET /questions"
definition: 플랜에 대한 Q&A 질문을 등록하거나 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
relatesTo:
  - to: concept.plan
    type: reads
    note: "질문은 반드시 하나의 플랜에 귀속된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

플랜 진행 과정에서 명확하지 않은 사항을 질문으로 남기고, 그 질문들을 목록으로 조회하는 Q&A 협업 기능이다. 에이전트가 스스로 판단하기 어려운 기획 모호성이나 제약 사항을 질문으로 기록해 두고, 이후 답변을 통해 의사결정 맥락을 남길 수 있다.

## 요청 / 응답

**POST /questions — 질문 생성**

요청 본문:

- `plan_id` — 질문이 속할 플랜 식별자. 필수.
- `body` — 질문 내용 텍스트. 필수.
- `asker` — 질문 작성자. 선택. 기본값은 `"agent"`.

새로 생성된 질문의 상태는 항상 `open` 으로 시작한다. `answer`, `answered_by`, `answered_at` 은 null 이다.

성공 응답: 생성된 질문 객체.

**GET /questions — 질문 목록 조회**

쿼리 파라미터:

- `plan_id` — 조회할 플랜 식별자. 필수.
- `status` — 선택. `open` 또는 `answered` 로 필터링한다. 생략하면 모든 상태의 질문을 반환한다.

지정된 플랜에 속한 질문 배열을 반환한다.

## 권한 / 제약

- `POST` 에서 `plan_id` 가 없거나 참조하는 플랜이 존재하지 않으면 404 오류를 반환한다.
- `GET` 에서 `plan_id` 쿼리 파라미터가 없으면 400 오류를 반환한다.
- 질문 상태는 `POST /questions/:id/answer` 를 통해서만 `answered` 로 전환된다. 이 엔드포인트에서 직접 상태를 바꿀 수 없다.

## provenance

기획 단계에서 에이전트가 모호한 사항을 구조화된 방식으로 기록하고 답변을 추적하는 요건에서 Q&A 기능이 설계되었다. 라우터는 `sdi-plugin/crates/daemon/src/router/collab.rs` 에 위치한다.

## 미확정 (OPEN)

- 하나의 플랜에 등록할 수 있는 질문 수에 제한이 있는지 확인되지 않았다.
- 질문이 특정 시나리오나 태스크에도 귀속될 수 있는지, 플랜 전용인지 확인이 필요하다. 현재 명세는 `plan_id` 만 받는다.
- `GET` 응답의 정렬 기준(생성 시각 오름차순/내림차순)이 명시되지 않았다.
- 페이지네이션 지원 여부가 확인되지 않았다.
