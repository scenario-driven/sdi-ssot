---
id: endpoint.questions-answer
kind: Endpoint
title: "POST /questions/:id/answer"
definition: Q&A 질문에 답변을 등록하고 질문 상태를 answered 로 전환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/collab.rs"
relatesTo:
  - to: concept.decision
    type: relates-to
    note: "질문 답변은 기획 모호성을 해소하는 역할을 하며, 의사결정 기록과 병렬적인 성격을 갖는다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

열린(open) 질문에 답변 텍스트를 등록한다. 답변이 등록되면 질문 상태가 `answered` 로 전환된다. 에이전트나 사람이 플랜 진행 중 발생한 모호성을 해소하고, 그 해소 맥락을 구조화된 방식으로 보존하는 데 사용한다.

## 요청 / 응답

**POST /questions/:id/answer**

경로 파라미터:

- `id` — 답변을 달 질문의 식별자. 필수.

요청 본문:

- `answer` — 답변 내용 텍스트. 필수.
- `by` — 답변 작성자. 선택. 기본값은 `"agent"`.

데몬은 답변 등록과 동시에 다음을 자동으로 처리한다.

1. 질문의 `answer` 필드를 요청 본문의 `answer` 로 채운다.
2. `answered_by` 를 요청 본문의 `by` 로 설정한다.
3. `answered_at` 을 현재 시각으로 기록한다.
4. 질문 `status` 를 `answered` 로 전환한다.

성공 응답: 갱신된 질문 객체(답변 포함 전체 필드). `question.answered` SSE 이벤트가 구독 중인 클라이언트에게 전파된다.

## 권한 / 제약

- 이미 `answered` 상태인 질문에 다시 답변을 시도하면 409 오류를 반환한다. 답변은 한 번만 등록할 수 있다.
- 질문이 존재하지 않으면 404 오류를 반환한다.
- `answer` 본문이 없거나 빈 문자열이면 400 오류를 반환한다.
- `answered_at` 은 서버가 자동으로 기록하며 요청 본문으로 지정할 수 없다.

## provenance

Q&A 답변 기능은 기획 단계의 모호성을 에이전트가 스스로 기록하고 해소할 수 있도록 설계되었다. 답변이 의사결정 기록(`concept.decision`)과 병렬적 성격을 갖는다는 점은, 질문 해소가 플랜의 방향을 실질적으로 바꿀 수 있다는 맥락에서 비롯된다. 라우터는 `sdi-plugin/crates/daemon/src/router/collab.rs` 에 위치한다.

## 미확정 (OPEN)

- 답변을 수정하거나 철회하는 엔드포인트가 별도로 존재하는지 확인이 필요하다.
- `question.answered` SSE 이벤트의 채널이 플랜 단위인지 전역 단위인지 명시되지 않았다.
- 답변이 등록된 이후 이 질문이 의사결정 기록(`decision`)으로 자동 변환되거나 연결되는 흐름이 존재하는지 확인이 필요하다.
- 동일 질문에 여러 답변을 허용하는 스레드형 구조로 확장될 가능성이 있는지 현재 설계에서는 확인되지 않았다.
