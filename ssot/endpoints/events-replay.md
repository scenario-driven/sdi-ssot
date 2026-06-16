---
id: endpoint.events-replay
kind: Endpoint
title: "GET /events/replay"
definition: >
  데몬의 이벤트 로그에 저장된 과거 SSE 이벤트를 JSON 배열로 재생한다.
  세션 중간에 합류한 클라이언트가 놓친 이벤트를 따라잡는 데 사용한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/aggregate.rs"
relatesTo:
  - to: concept.dispatch
    type: reads
    note: 데몬이 발행한 이벤트 로그를 읽어 재생한다
  - to: concept.project
    type: reads
    note: project_id 파라미터로 특정 프로젝트 이벤트만 걸러낼 수 있다
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

데몬이 과거에 발행한 SSE(Server-Sent Events) 이벤트를 이벤트 로그에서 읽어 JSON 배열로 돌려주는 엔드포인트다. `GET /events` 가 실시간 스트림만 제공하는 것과 달리, 이 엔드포인트는 이미 지나간 이벤트를 되감아 보여준다. 클라이언트가 연결을 잠시 끊었다가 재연결할 때 그 사이 놓친 이벤트를 따라잡는 상황에 적합하다. 응답은 SSE 스트림이 아니라 일반 JSON 배열이다.

## 요청 / 응답

**요청**

- 메서드: `GET`
- 경로: `/events/replay`
- 쿼리 파라미터
  - `project_id` (선택): 특정 프로젝트의 이벤트만 반환한다. 생략하면 모든 프로젝트의 이벤트를 대상으로 한다.
  - `since` (선택): RFC 3339 형식의 타임스탬프. 이 시각 이후에 발생한 이벤트만 반환한다. 생략하면 최신 이벤트부터 limit 수만큼 반환한다.
  - `limit` (선택): 반환할 최대 이벤트 수. 기본값은 200이다.

**응답**

- 성공 시 HTTP 200
- 본문: 이벤트 객체의 JSON 배열. SSE 스트림 형식이 아니다.
- 조건에 맞는 이벤트가 없으면 빈 배열 `[]` 을 반환한다.

## 권한 / 제약

- 읽기 전용 작업이므로 데이터를 변경하지 않는다.
- 데몬이 실행 중이어야 응답을 받을 수 있다.
- `limit` 기본값이 200이므로, 로그가 길면 `since` 파라미터를 함께 사용해 범위를 좁히는 것이 바람직하다.

## provenance

- 라우트 출처: `sdi-plugin/crates/daemon/src/router/aggregate.rs`
- 실시간 이벤트 구독은 `GET /events` 가 담당하며, 이 엔드포인트는 과거 이벤트 보완 조회 전용이다.

## 미확정 (OPEN)

- 데몬이 이벤트 로그를 얼마나 오래 보관하는지(보존 기간·최대 크기) 명시되지 않았다.
- `since` 를 생략했을 때 기준점(가장 오래된 이벤트부터인지, 최신 limit 개인지)이 명확하지 않다.
- 이벤트 객체의 정확한 필드 구조(타입, 페이로드 형식 등)가 명세에 정의되지 않았다.
- 인증·권한 검사 여부가 명시되지 않았다.
