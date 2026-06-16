---
id: endpoint.plans
kind: Endpoint
title: "POST /plans, GET /plans"
definition: 플랜을 생성하거나 프로젝트 단위로 목록을 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
relatesTo:
  - to: concept.plan
    type: mutates
    note: "POST로 새 플랜을 draft 상태로 생성한다"
  - to: concept.project
    type: reads
    note: "GET의 project_id 필터 및 POST의 소속 프로젝트 지정에 사용된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

SDI 플랜을 생성하고 목록을 조회하는 엔드포인트 묶음이다. 플랜은 하나의 프로젝트 안에서 구현 목표와 시나리오 묶음을 구조화하는 단위다. 새로 만들어진 플랜은 항상 초안(draft) 상태로 시작하며, 승인(active 전환) 전까지는 태스크를 시작할 수 없다. 생성 시 SSE(서버 전송 이벤트)로 `plan.created` 이벤트가 발행된다.

## 요청 / 응답

**POST /plans — 플랜 생성**

- 요청 본문:
  - `project_id` (필수): 이 플랜이 속할 프로젝트의 고유 식별자
  - `short_code` (필수): 플랜을 짧게 구분하는 코드 (예: `P1`, `SPRINT-3`)
  - `title` (필수): 플랜의 제목
  - `body` (선택): 플랜의 상세 설명이나 목표 기술 (마크다운 가능)
- 응답: 생성된 플랜 레코드를 JSON으로 반환한다. 상태는 `draft`로 고정된다.
- 부수 효과: `plan.created` SSE 이벤트가 발행되어 구독 중인 클라이언트에 변경이 전달된다.

---

**GET /plans — 플랜 목록 조회**

- 쿼리 파라미터:
  - `project_id` (필수): 조회 대상 프로젝트의 고유 식별자
- 응답: 해당 프로젝트에 속한 플랜 목록을 JSON 배열로 반환한다.

## 권한 / 제약

- 인증이 필요하지 않다. 데몬 소켓에 접근 가능한 로컬 프로세스만 호출할 수 있다.
- 플랜 생성 시 `short_code`는 같은 프로젝트 안에서 고유해야 한다(고유성 보장 여부는 미확정).
- 새로 생성된 플랜은 반드시 `draft` 상태에서 시작한다. 생성 시점에 `active` 상태를 직접 지정할 수 없다.
- GET에서 `project_id`가 없으면 오류를 반환한다(전체 플랜 목록 조회는 지원하지 않는 것으로 추정).

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/plan.rs`
- 이 엔드포인트의 존재와 동작은 코드 탐색 없이 설계 문서 기반으로 추론되었으므로 confidence가 inferred다.

## 미확정 (OPEN)

- `short_code`의 프로젝트 내 고유성을 데몬이 강제하는지, 아니면 호출자가 보장해야 하는지 미확정이다.
- GET에서 상태(`draft` / `active` / `completed`)별 필터링 옵션 지원 여부가 미확정이다.
- `body` 필드의 최대 길이 제한 여부가 미확정이다.
- `plan.created` SSE 이벤트의 페이로드 형식(어떤 필드가 포함되는지)이 미확정이다.
- 존재하지 않는 `project_id`로 POST 요청 시 오류 응답 형식이 미확정이다.
