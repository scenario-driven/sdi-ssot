---
id: endpoint.plan
kind: Endpoint
title: "GET /plans/:id, PUT /plans/:id"
definition: 특정 플랜을 ID로 조회하거나, 플랜의 제목과 본문을 갱신한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
relatesTo:
  - to: concept.plan
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

플랜 단건 조회(GET)와 플랜 내용 갱신(PUT)을 담당하는 엔드포인트다. GET은 ID에 해당하는 플랜 한 건을 반환한다. PUT은 플랜의 제목과 본문을 새 값으로 교체하고, 호출할 때마다 플랜의 버전 카운터를 1씩 올린다.

## 요청 / 응답

**GET /plans/:id**
- 경로 파라미터 `id`: 조회할 플랜의 고유 식별자.
- 응답: 플랜 객체(id, 제목, 본문, 상태, 버전, 생성/수정 시각 등).
- 플랜이 존재하지 않으면 404를 반환한다.

**PUT /plans/:id**
- 경로 파라미터 `id`: 갱신할 플랜의 고유 식별자.
- 요청 본문(JSON): `title`(필수)과 `body`(필수) 두 필드를 모두 포함해야 한다. 하나라도 빠지면 요청을 거부한다.
- 응답: 갱신된 플랜 객체(버전 카운터가 올라간 상태).
- 갱신 성공 시 `plan.updated` SSE 이벤트를 발행한다.

## 권한 / 제약

- PUT은 제목과 본문을 **동시에** 전달해야 한다. 부분 갱신(PATCH 방식)은 지원하지 않는다.
- 버전 카운터는 서버가 자동으로 관리하며 클라이언트가 직접 지정할 수 없다.
- 상태 전환(draft→active 등)은 이 엔드포인트의 역할이 아니다. 승인은 `/plans/:id/approve`, 완료는 `/plans/:id/complete`를 사용한다.

## provenance

sdi-plugin 플랜 라우터(`plan.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. 라우터 파일 경로와 필드 목록은 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- PUT 시 플랜 상태(active/completed)와 무관하게 본문 갱신이 허용되는지, 아니면 특정 상태에서만 가능한지 확인 필요.
- 버전 카운터의 초기값과 최대값 정책 미확인.
- `plan.updated` SSE 이벤트의 페이로드 스키마 미확인.
