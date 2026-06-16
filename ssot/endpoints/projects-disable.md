---
id: endpoint.projects-disable
kind: Endpoint
title: "POST /projects/:id/disable"
definition: 프로젝트를 비활성화하여 해당 프로젝트의 모든 훅 강제를 일시 중단한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.project
    type: mutates
    note: "프로젝트의 enabled 필드를 false로 변경한다"
  - to: concept.autonomy-policy
    type: relates-to
    note: "비활성화 시 해당 프로젝트 cwd에 대한 훅 게이트가 모두 우회된다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

지정한 프로젝트를 비활성화 상태로 전환하는 엔드포인트다. 비활성화된 프로젝트의 경우, 플러그인 훅 레이어가 해당 프로젝트의 작업 디렉토리에서 일어나는 파일 변경·도구 실행 등에 대한 SDI 게이트 검사를 건너뛴다. 일시적으로 SDI 강제를 멈추고 싶을 때 사용한다.

## 요청 / 응답

**요청**

- 메서드: POST
- 경로: `/projects/:id/disable`
- 경로 파라미터:
  - `id` (필수): 비활성화할 프로젝트의 고유 식별자
- 요청 본문: 없음

**응답**

- 성공: 업데이트된 프로젝트 레코드(enabled=false 포함)를 JSON으로 반환하거나, 성공을 나타내는 빈 응답을 반환한다.
- 존재하지 않는 프로젝트 id일 경우 404를 반환한다.

## 권한 / 제약

- 인증이 필요하지 않다. 데몬 소켓에 접근 가능한 로컬 프로세스만 호출할 수 있다.
- 이미 비활성화된 프로젝트에 대해 다시 호출해도 오류 없이 처리된다(멱등성 여부는 미확정).
- 비활성화 이후에도 프로젝트 레코드와 데이터는 삭제되지 않고 보존된다.
- 훅 우회가 즉시 적용되는지, 아니면 다음 훅 실행 시점에 반영되는지는 미확정이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/project.rs`
- 이 엔드포인트의 존재와 동작은 코드 탐색 없이 설계 문서 기반으로 추론되었으므로 confidence가 inferred다.

## 미확정 (OPEN)

- 비활성화 시 SSE(서버 전송 이벤트)를 통해 상태 변경을 알리는지 미확정이다.
- 비활성화된 프로젝트에 속한 활성 태스크나 진행 중인 라운드의 처리 방식이 미확정이다.
- 멱등성 보장 여부(이미 disabled인 프로젝트에 재호출 시 응답)가 미확정이다.
