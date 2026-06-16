---
id: endpoint.projects-cwds
kind: Endpoint
title: "POST /projects/:id/cwds, DELETE /projects/:id/cwds, GET /projects/:id/cwds"
definition: 프로젝트에 연결된 작업 디렉토리(cwd) 목록을 관리한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/project.rs"
relatesTo:
  - to: concept.project
    type: mutates
    note: "프로젝트의 cwd 목록을 추가·삭제·조회한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

프로젝트 하나에 연결된 작업 디렉토리(cwd) 목록을 관리하는 엔드포인트 묶음이다. 한 프로젝트는 여러 cwd를 가질 수 있으며, 훅 레이어는 이 목록을 통해 현재 열린 폴더가 어느 프로젝트에 속하는지 판단한다. 새 디렉토리를 프로젝트에 붙이거나(POST), 기존 연결을 끊거나(DELETE), 현재 목록을 확인(GET)할 때 사용한다.

## 요청 / 응답

**공통 경로 파라미터**

- `id` (필수): 대상 프로젝트의 고유 식별자

---

**POST /projects/:id/cwds — cwd 추가**

- 요청 본문: `{ "cwd": "<절대 경로 문자열>" }`
- 응답: 추가된 cwd 항목 또는 업데이트된 cwd 전체 목록을 반환한다.

---

**DELETE /projects/:id/cwds — cwd 제거**

- 요청 본문: `{ "cwd": "<절대 경로 문자열>" }` 또는 쿼리 파라미터로 전달 (미확정)
- 응답: 제거 완료 후의 cwd 목록 또는 성공을 나타내는 빈 응답을 반환한다.

---

**GET /projects/:id/cwds — cwd 목록 조회**

- 요청 본문: 없음
- 응답: 프로젝트에 현재 등록된 cwd 경로 목록을 JSON 배열로 반환한다.

## 권한 / 제약

- 인증이 필요하지 않다. 데몬 소켓에 접근 가능한 로컬 프로세스만 호출할 수 있다.
- 하나의 cwd 경로는 복수의 프로젝트에 동시에 등록되지 않아야 한다. 중복 등록에 대한 오류 처리 방식은 미확정이다.
- 존재하지 않는 경로를 등록하는 것을 막는 유효성 검사 여부는 미확정이다.

## provenance

- 라우터 파일: `sdi-plugin/crates/daemon/src/router/project.rs`
- 이 엔드포인트의 존재와 동작은 코드 탐색 없이 설계 문서 기반으로 추론되었으므로 confidence가 inferred다.

## 미확정 (OPEN)

- DELETE 요청에서 cwd를 요청 본문으로 전달하는지, 쿼리 파라미터로 전달하는지 미확정이다.
- cwd 경로가 심볼릭 링크를 포함할 때 정규화(resolve) 여부가 미확정이다.
- cwd를 제거한 뒤 해당 경로에서 진행 중이던 태스크 또는 훅의 처리 방식이 미확정이다.
- cwd 추가 시 중복 등록(이미 같은 프로젝트에 동일 경로가 있을 때)의 응답 동작이 미확정이다.
