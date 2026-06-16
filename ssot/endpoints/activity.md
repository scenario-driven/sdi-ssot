---
id: endpoint.activity
kind: Endpoint
title: "GET /activity, POST /activity"
definition: 프로젝트 내에서 발생한 모든 활동 이벤트를 추가 전용으로 기록하고 조회하는 엔드포인트. 이벤트는 한 번 기록되면 수정하거나 삭제할 수 없다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/collab.rs
relatesTo:
  - to: concept.project
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

프로젝트 안에서 일어난 모든 활동(에이전트 행위, 사용자 조작, 시스템 이벤트 등)을 추가 전용 로그로 쌓는 엔드포인트다. POST 로 새 이벤트를 기록하고, GET 으로 기록된 이벤트 목록을 조회한다. 이미 기록된 이벤트에 대해 PATCH 나 DELETE 를 호출하면 405(허용하지 않는 메서드)를 반환한다. SSE(서버 전송 이벤트)는 발행하지 않으며, 순수하게 감사 로그 목적으로 동작한다.

## 요청 / 응답

**POST /activity** — 새 활동 이벤트 기록
- 요청 본문: `project_id`(필수), `kind`(이벤트 종류, 필수), `summary`(요약 문자열, 필수), `actor`(행위자, 기본값 "agent"), `entity_id`(관련 엔티티 ID, 선택), `payload`(임의 JSON 데이터, 선택)
- 응답: 생성된 이벤트 객체(id, project_id, kind, summary, actor, entity_id, payload, created_at)

**GET /activity** — 활동 이벤트 목록 조회
- 쿼리 파라미터: `project_id`(필수), `limit`(반환 최대 건수, 기본값 100)
- 응답: 이벤트 배열, 최신 이벤트 우선 정렬

**PATCH /activity/:id, DELETE /activity/:id** — 405 Method Not Allowed 반환 (추가 전용 불변 정책)

## 권한 / 제약

- `project_id` 없이 GET 을 호출하면 400 오류를 반환한다.
- 이벤트는 한 번 기록되면 변경하거나 삭제할 수 없다(추가 전용 불변 원칙).
- SSE 이벤트를 발행하지 않으므로 실시간 구독 목적으로는 사용할 수 없다.

## provenance

collab.rs 라우터에서 구현된다는 정보는 라우터 파일 목록 분석을 통해 추론되었다.

## 미확정 (OPEN)

- `kind` 필드가 허용하는 이벤트 종류 목록이 열거형으로 고정되어 있는지, 자유 문자열인지 미확인.
- `limit` 의 상한값(최대 허용 건수)이 별도로 제한되는지 미확인.
- 특정 `kind` 또는 `actor` 기준 필터링 쿼리 파라미터가 추가로 지원되는지 미확인.
