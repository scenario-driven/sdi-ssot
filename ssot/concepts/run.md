---
id: concept.run
kind: Concept
title: Run(실행 기록)
definition: "태스크의 단일 실행 시도 기록. 하나의 태스크에 여러 Run이 생길 수 있다(재시도, 병렬 fan-out). 에이전트 이름(actor), 세션 ID, 시작/종료 시각, 결과(success/failure/aborted), 자유 메모와 페이로드를 담는다."
relatesTo:
  - { to: concept.task, type: belongs-to, note: "Run은 태스크에 속하며 태스크의 실행 이력을 구성한다." }
  - { to: concept.usage-budget, type: relates-to, note: "Run과 UsageRecord는 run_id로 연결되어 실행별 토큰/비용을 추적한다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/run.rs
  - sdi-plugin/crates/db/src/migrations/004_runs_hierarchy.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Run(실행 기록)은 에이전트가 태스크를 실행하는 단일 시도를 기록한 것이다. 태스크가 여러 번 실행될 수 있고(재시도, 서브에이전트 병렬 실행), 각 실행이 하나의 Run 행을 만든다. Run은 태스크의 status 전이와 독립적이다 — 태스크가 `in_progress`인 상태에서 여러 Run이 동시에 또는 순차적으로 존재할 수 있다.

Run의 결과(result)는 세 가지다.

- **success**: 실행이 성공적으로 완료됨.
- **failure**: 실행이 실패함.
- **aborted**: 실행이 중단됨(내부 오류, 외부 취소 등).

`result`는 nullable이다. 실행이 진행 중이면 `finished_at`과 `result` 모두 NULL이다.

`session_id`는 어떤 Claude Code 세션에서 이 Run이 시작되었는지 추적한다. 다중 세션 협업(D29)에서 어느 세션이 어떤 실행을 했는지 파악하는 데 사용된다.

## 엔티티 (DB)

Run 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 태스크 참조.
- **실행자**: actor(에이전트 이름 또는 사용자), session_id(optional).
- **시각**: started_at, finished_at(optional).
- **결과**: result(success/failure/aborted, optional — 진행 중이면 NULL).
- **메모**: notes(자유 서술 메모), payload(JSON 자유 확장).

## API 표면

Run은 태스크 실행 시 훅 또는 에이전트가 생성한다. 조회는 태스크 조회 결과에 포함되거나 별도 Run 목록 API로 가져온다.

## 불변식

- **시작 시각 필수**: started_at은 항상 기록된다.
- **결과 nullable**: 진행 중인 Run은 result가 NULL이다.

## 구현 위치 (provenance)

Run, TaskRelation, TaskLease 구조체는 `sdi-plugin/crates/core/src/run.rs`에 있다. DB 스키마는 마이그레이션 004에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: SubagentStart/SubagentStop 훅이 자동으로 Run을 생성하는지 확인 필요.
- [ ] OPEN: TaskLease(단일-쓰기 보호)와 Run의 관계 — 동시에 여러 Run이 실행될 때 TaskLease가 어떻게 조정되는지 확인 필요.
