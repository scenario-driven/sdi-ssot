---
id: endpoint.projects-plans-active
kind: Endpoint
title: "GET /projects/:project_id/plans/active"
definition: 특정 프로젝트에서 현재 활성 상태인 플랜 한 건을 반환한다. 없으면 null을 반환한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
relatesTo:
  - to: concept.plan
    type: reads
  - to: concept.project
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

프로젝트별로 현재 진행 중인 플랜이 무엇인지를 알려주는 엔드포인트다. 한 프로젝트에 활성 플랜은 최대 1개뿐이라는 제약에 따라, 이 경로는 항상 단건 또는 null을 반환한다. 대시보드와 CLI가 현재 작업 컨텍스트를 파악할 때 이 엔드포인트를 기준으로 삼는다.

## 요청 / 응답

**GET /projects/:project_id/plans/active**
- 경로 파라미터 `project_id`: 조회할 프로젝트의 고유 식별자.
- 응답: 활성 플랜 객체, 또는 활성 플랜이 없으면 `null`(빈 응답이 아니라 명시적 null).
- 프로젝트 자체가 존재하지 않으면 404를 반환한다.

## 권한 / 제약

- 프로젝트당 동시에 활성 상태인 플랜은 최대 1개다. 이 제약은 플랜 승인(`/plans/:id/approve`) 단계에서 강제된다.
- 조회 전용(읽기 전용) 엔드포인트다. 상태를 변경하지 않는다.
- 대시보드와 CLI 모두 이 경로를 통해 현재 작업 컨텍스트(어느 플랜 아래서 일하고 있는지)를 해소한다.

## provenance

sdi-plugin 플랜 라우터(`plan.rs`)에서 구현한다고 명시되어 있으나, 실제 소스 파일을 직접 확인하지 않은 상태다. 단건/null 반환 정책과 사용 주체(대시보드·CLI)는 설계 문서 기반으로 기입하였다.

## 미확정 (OPEN)

- 활성 플랜이 없을 때 HTTP 200 + null 바디로 반환하는지, 아니면 404로 반환하는지 미확인.
- 응답 객체에 포함되는 필드 목록(전체 플랜 객체인지 요약 형태인지) 미확인.
- 동시에 활성 플랜이 복수로 존재하는 데이터 오류 상황에 대한 방어 처리 여부 미확인.
