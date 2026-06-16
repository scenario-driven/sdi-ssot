---
id: endpoint.plans-import
kind: Endpoint
title: "POST /plans/import"
definition: /plans/export 로 내보낸 번들을 가져와서 엔티티를 최선-노력 업서트(upsert) 방식으로 복원하는 엔드포인트. 이미 존재하는 엔티티는 조용히 건너뛴다. 백업 복원 및 프로젝트 간 마이그레이션에 사용한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.plan
    type: mutates
    note: ""
  - to: concept.scenario
    type: mutates
    note: ""
  - to: concept.requirement
    type: mutates
    note: ""
  - to: concept.decision
    type: mutates
    note: ""
  - to: concept.round
    type: mutates
    note: ""
  - to: concept.task
    type: mutates
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

`/plans/export` 로 내보낸 번들을 역방향으로 가져와서 각 엔티티를 최선-노력(best-effort) 업서트 방식으로 저장한다. 이미 같은 ID 로 존재하는 엔티티는 오류 없이 조용히 건너뛴다(skip). 완료 후 엔티티 유형별 가져온 건수를 반환한다. 백업 복원 및 프로젝트 간 마이그레이션 두 가지 목적으로 모두 사용할 수 있다.

## 요청 / 응답

**POST /plans/import**
- 요청 본문:
  - `project_id`(선택): 가져올 대상 프로젝트 ID. 지정하면 번들 내 원래 project_id 를 덮어쓴다.
  - `plans`(필수): 내보내기 번들과 동일한 구조의 계획 배열. 각 계획은 plan, requirements, decisions, scenarios, rounds, tasks 를 포함.
- 응답: 엔티티 유형별 가져온 건수 객체
  - 예: `{ "plans": 2, "scenarios": 15, "requirements": 8, "decisions": 4, "rounds": 6, "tasks": 20, "skipped": 3 }`

## 권한 / 제약

- `plans` 배열이 없거나 빈 배열이면 400 오류를 반환한다.
- 이미 존재하는 엔티티(동일 ID)는 덮어쓰지 않고 건너뛴다. 강제 덮어쓰기 옵션은 현재 명세상 지원하지 않는다.
- 트랜잭션 처리 방식(전체 성공/전체 실패 vs 부분 성공)이 현재 명세상 불명확하다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 업서트 의미론, 응답 구조는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- 가져오기 도중 하나의 엔티티 처리가 실패했을 때 전체를 롤백하는지, 성공한 엔티티만 커밋하는지(부분 성공) 미확인.
- `project_id` 를 덮어쓸 경우 번들 내 엔티티 간 참조(예: 태스크의 plan_id)가 자동으로 업데이트되는지 미확인.
- SSE 이벤트 발행 여부 미확인.
