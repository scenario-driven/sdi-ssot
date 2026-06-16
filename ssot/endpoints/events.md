---
id: endpoint.events
kind: Endpoint
title: "GET /events"
definition: 데몬(sdid)이 발행하는 모든 도메인 이벤트를 실시간으로 전달하는 Server-Sent Events(SSE) 스트림 엔드포인트.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/events.rs"
relatesTo:
  - to: concept.dispatch
    type: reads
    note: "데몬 내부 이벤트 브로커(dispatch)에서 발행된 이벤트를 구독하여 HTTP 스트림으로 중계한다."
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

데몬이 내부에서 생성하는 도메인 이벤트를 외부 구독자(대시보드 SPA, CLI 훅)에게 실시간으로 푸시한다. 표준 SSE 프로토콜(HTTP/1.1 chunked, `text/event-stream`)을 사용하므로 구독자는 일반 HTTP 연결만으로 이벤트를 수신할 수 있다. 연결이 끊기면 클라이언트가 재연결하는 방식으로 동작하며, 데몬은 연결된 구독자 수에 관계없이 같은 이벤트를 브로드캐스트한다.

## 요청 / 응답

요청 파라미터 없음. `Accept: text/event-stream` 헤더를 포함하여 연결한다.

응답은 끊기지 않는 스트림이다. 각 이벤트는 SSE 표준 형식(`event:`, `data:`, `id:` 필드)으로 전달된다. 현재 알려진 이벤트 유형은 다음을 포함한다.

- **scenario.created** — 새 시나리오가 등록되었을 때.
- **plan.approved** — 플랜이 승인(active)되었을 때.
- **round.activated** — 라운드가 활성화되었을 때.
- **task.completed** — 태스크가 완료(done/cancelled) 상태로 전환되었을 때.

이벤트 유형 목록은 데몬이 내부적으로 dispatch하는 전체 이벤트 집합을 반영하며, 구현이 확장될수록 늘어날 수 있다. 각 이벤트의 `data` 필드에는 관련 도메인 객체의 핵심 식별자(id, 이름 등)가 JSON으로 포함된다.

## 권한 / 제약

인증 불필요 (로컬 데몬 접근 기준). 스트림은 읽기 전용이며 데이터를 변경하지 않는다. 구독자가 많아질수록 연결 유지 오버헤드가 증가하므로 단일 대시보드 SPA 기준 설계가 기본 전제다. 연결 유지 시간(keep-alive) 정책은 미확인.

## provenance

- 대시보드 SPA가 초기 로드 후 이 엔드포인트에 연결하여 UI를 실시간으로 갱신한다.
- CLI가 특정 작업의 완료를 기다리는 폴링 대신 이 스트림을 구독할 수 있다.
- 이벤트 브로커 구현 위치: `sdi-plugin/crates/daemon/src/events.rs`.

## 미확정 (OPEN)

- 이벤트의 전체 목록과 각 이벤트의 `data` 페이로드 스키마가 공식 문서화되지 않음.
- `id:` 필드를 활용한 재연결 시 누락 이벤트 replay 지원 여부 미확인.
- 프로젝트 범위 필터(특정 프로젝트의 이벤트만 구독) 쿼리 파라미터 지원 여부 미확인.
- 연결 유지 heartbeat(ping) 이벤트 발행 주기 미확인.
