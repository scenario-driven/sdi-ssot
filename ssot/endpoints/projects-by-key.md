---
id: endpoint.projects-by-key
kind: Endpoint
title: "GET /projects/by-key/:key"
definition: 티켓 접두어(key)로 프로젝트를 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.ticket-number
    type: reads
    note: "티켓 번호의 접두어(key)를 기준으로 프로젝트를 찾는다"
  - to: concept.project
    type: reads
    note: "key에 해당하는 프로젝트 레코드 전체를 반환한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

프로젝트마다 고유하게 할당된 티켓 접두어(예: `LM`, `SDI`, `CK`)를 경로 파라미터로 받아, 해당 프로젝트 레코드를 반환하는 엔드포인트다. CLI에서 사용자가 짧은 키로 프로젝트를 지정할 때, 내부적으로 전체 프로젝트 레코드로 변환하기 위해 사용된다.

## 요청 / 응답

**요청**

- 메서드: GET
- 경로: `/projects/by-key/:key`
- 경로 파라미터:
  - `key` (필수): 프로젝트에 지정된 티켓 접두어 문자열. 예: `LM`, `SDI`, `CK`

**응답**

- 성공: 해당 key를 가진 프로젝트의 전체 레코드를 JSON으로 반환한다.
- 일치하는 프로젝트가 없으면 404를 반환한다.

## 권한 / 제약

- 인증이 필요하지 않다. 데몬 소켓에 접근 가능한 로컬 프로세스만 호출할 수 있다.
- 조회만 수행하며 데이터를 변경하지 않는다.
- 티켓 접두어는 프로젝트 간 중복될 수 없다. key는 전역 고유값이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/project.rs`
- 이 엔드포인트의 존재와 동작은 코드 탐색 없이 설계 문서 기반으로 추론되었으므로 confidence가 inferred다.

## 미확정 (OPEN)

- key 비교가 대소문자 구분을 하는지 여부가 미확정이다.
- 비활성화된 프로젝트(enabled=false)도 결과에 포함되는지 미확정이다.
- key 형식(허용 문자, 최대 길이)에 대한 유효성 검사 규칙이 미확정이다.
