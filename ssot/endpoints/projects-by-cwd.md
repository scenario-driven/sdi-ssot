---
id: endpoint.projects-by-cwd
kind: Endpoint
title: "GET /projects/by-cwd"
definition: 주어진 작업 디렉토리 경로를 포함하는 프로젝트를 조회한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.project
    type: reads
    note: "cwd 목록에 해당 경로가 포함된 프로젝트 레코드를 반환한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

현재 작업 중인 디렉토리 경로를 받아, 그 경로가 등록된 프로젝트를 찾아 반환하는 엔드포인트다. Claude Code 플러그인의 훅(hook) 레이어가 실행되는 시점에, 현재 열려 있는 폴더가 어느 SDI 프로젝트에 속하는지 빠르게 판단하기 위해 사용된다.

## 요청 / 응답

**요청**

- 메서드: GET
- 경로: `/projects/by-cwd`
- 쿼리 파라미터:
  - `cwd` (필수): 현재 작업 디렉토리의 절대 경로 문자열. 예: `/Users/alice/dev/my-project`

**응답**

- 성공: 해당 경로를 포함하는 프로젝트의 전체 레코드를 JSON으로 반환한다.
- 해당 경로를 포함하는 프로젝트가 없으면 404 또는 빈 결과를 반환한다.

## 권한 / 제약

- 인증이 필요하지 않다. 데몬 소켓에 접근 가능한 로컬 프로세스만 호출할 수 있다.
- 조회만 수행하며 데이터를 변경하지 않는다.
- 하나의 cwd 경로는 최대 하나의 프로젝트에만 귀속될 수 있다. 복수 매칭이 발생하는 경우 동작은 미확정이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/project.rs`
- 이 엔드포인트의 존재와 동작은 코드 탐색 없이 설계 문서 기반으로 추론되었으므로 confidence가 inferred다.

## 미확정 (OPEN)

- `cwd` 파라미터가 등록된 경로의 하위 디렉토리일 때 prefix 매칭을 수행하는지, 정확 일치만 하는지 미확정이다.
- 비활성화된 프로젝트(enabled=false)도 결과에 포함되는지 미확정이다.
- 복수의 cwd가 동일 경로에 매칭되는 충돌 케이스의 오류 응답 형식이 미확정이다.
