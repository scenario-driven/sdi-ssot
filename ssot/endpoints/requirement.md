---
id: endpoint.requirement
kind: Endpoint
title: "GET /requirements/:id, PUT /requirements/:id, DELETE /requirements/:id"
definition: 요구사항 단건을 조회하거나, 내용을 교체하거나, 삭제한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/requirement.rs"
relatesTo:
  - to: concept.requirement
    type: mutates
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

요구사항 단건 조작 엔드포인트다. GET으로 특정 요구사항 하나를 가져오고, PUT으로 제목·본문·출처를 새 값으로 교체하며, DELETE로 요구사항을 제거한다. PUT은 스냅샷 전용 원칙을 따른다. 변경 이력은 본문이 아니라 결정(decision) 기록에 따로 남긴다.

## 요청 / 응답

**GET /requirements/:id**
- 경로 파라미터 `id`: 조회할 요구사항의 고유 식별자.
- 응답: 요구사항 객체(id, short_code, title, body, source, plan_id, 생성/수정 시각 등).
- 존재하지 않으면 404를 반환한다.

**PUT /requirements/:id**
- 경로 파라미터 `id`: 갱신할 요구사항의 고유 식별자.
- 요청 본문(JSON): `title`, `body`, `source` 를 새 값으로 전달한다. 스냅샷 전용 형식이어야 하며, 본문에 변경 이력 흔적을 포함하면 거부한다.
- 응답: 갱신된 요구사항 객체.
- 갱신 성공 시 `requirement.updated` 이벤트를 발행한다.

**DELETE /requirements/:id**
- 경로 파라미터 `id`: 삭제할 요구사항의 고유 식별자.
- 응답: 삭제 확인 응답(본문 없음 또는 확인 메시지).
- 삭제 성공 시 `requirement.deleted` 이벤트를 발행한다.

## 권한 / 제약

- PUT은 스냅샷 전용(SNAPSHOT-ONLY) 원칙을 적용한다. 본문 안에 "이전에는 A였으나 지금은 B다"와 같은 변경 이력을 포함하면 요청을 거부한다. 이전 내용에 대한 기록이 필요하면 별도 결정(decision) 항목에 남긴다.
- DELETE는 복구가 불가능하므로 신중하게 호출해야 한다. 연결된 시나리오나 태스크가 있는 상태에서 삭제 허용 여부는 미확정이다.

## provenance

sdi-plugin 요구사항 라우터(`requirement.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. 스냅샷 검증 동작과 이벤트명은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- PUT 시 `title`, `body`, `source` 모두 필수인지, 일부만 전달해도 되는지(부분 갱신 허용 여부) 미확인.
- DELETE 시 연결된 시나리오나 태스크가 있을 때 차단하는지, cascade 삭제하는지, 고아 처리하는지 미확인.
- `requirement.updated`, `requirement.deleted` 이벤트가 SSE로 발행되는지 아니면 다른 채널인지 미확인.
- 이벤트 페이로드 스키마 미확인.
