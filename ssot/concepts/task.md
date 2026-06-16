---
id: concept.task
kind: Concept
title: Task(태스크)
definition: "LLM이 라운드 안에서 실행하는 원자적 작업 단위. 시나리오와 요건을 분해해 LLM이 생성하는 런타임 산물이며 인간이 직접 작성하지 않는다(D3). todo → in_progress → done | cancelled | blocked 상태 기계를 따르며, done 전환에는 시나리오 증거(Evidence)가 필수다."
relatesTo:
  - { to: concept.round, type: belongs-to, note: "태스크는 라운드에 속하며, 라운드를 통해 플랜에 간접 소속된다." }
  - { to: concept.scenario, type: relates-to, note: "태스크는 parent_scenario_ids로 자신이 어느 시나리오에서 비롯되었는지 가리킨다." }
  - { to: concept.requirement, type: relates-to, note: "태스크는 parent_requirement_ids로 관련 요건을 가리킨다." }
  - { to: concept.task-evidence, type: relates-to, note: "done 전환 시 시나리오별 판정 결과와 근거 참조를 담은 Evidence가 필수다." }
  - { to: concept.ticket-number, type: relates-to, note: "태스크는 플랜 내 고유 short_code(예: T-1)와 시스템 ID(TASK-...)를 함께 갖는다." }
  - { to: concept.run, type: relates-to, note: "태스크가 실행될 때마다 Run 기록이 생성된다. 재시도·병렬 실행 시 여러 Run이 하나의 태스크에 연결된다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "태스크는 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: domain.task-decomposition, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/task.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/010_short_code_scope.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Task(태스크)는 SDI에서 LLM이 직접 실행하는 가장 작은 작업 단위다. 중요한 점은 태스크는 **LLM이 시나리오·요건을 분해해 만드는 런타임 산물**이지 인간이 미리 정의하는 계획 항목이 아니라는 것이다(D3). 이 때문에 SDI는 Clawket의 Task 개념과 다르다 — Clawket에서는 인간이 직접 태스크를 작성하지만, SDI에서 그 역할은 시나리오가 담당한다.

태스크의 생애주기는 다음과 같다.

- **todo**: 생성은 되었으나 아직 시작 전.
- **in_progress**: 에이전트가 실행 중.
- **done**: 완료. 시나리오 증거(Evidence)가 반드시 첨부되어야 한다.
- **cancelled**: 취소됨.
- **blocked**: 외부 의존이나 다른 태스크에 의해 막힌 상태.

허용된 상태 전이는 다음과 같다: todo→in_progress, todo→cancelled, todo→blocked, in_progress→done, in_progress→cancelled, in_progress→blocked, blocked→todo, blocked→in_progress, blocked→cancelled. 완료된 태스크를 다시 in_progress로 되돌리는 전이는 허용되지 않는다.

태스크는 플랜에 직접 소속되지 않고 라운드를 통해 간접 소속된다. 다만 성능과 티켓 번호 계약을 위해 plan_id가 태스크에 비정규화되어 저장된다.

## 엔티티 (DB)

태스크 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 라운드 참조, 비정규화된 플랜 참조(태스크 생성 시 라운드에서 복사됨. 변경되지 않음).
- **식별**: 플랜 내 고유 short_code(예: T-1), 설명.
- **상태**: 현재 상태(todo/in_progress/done/cancelled/blocked).
- **부모 참조**: 이 태스크를 낳은 시나리오 ID 목록, 요건 ID 목록.
- **증거**: done 전환 시 필수인 시나리오별 판정 결과와 근거(Evidence). 채워지면 evidence_at 타임스탬프도 기록된다.
- **패턴 출처(D23)**: produced_via_pattern_id — 어떤 협업 패턴 아래 만들어졌는지.
- **타임스탬프**: 생성, 수정 시각.

## API 표면

태스크는 라운드 실행 중 에이전트가 생성하며, 사람은 보통 직접 생성하지 않는다. 상태 전이(complete, cancel, block, unblock)는 각각 별도 API로 이루어지며, done 전환 시에는 Evidence를 함께 전달해야 한다. 미 전달 또는 빈 Evidence로 done을 시도하면 `EVIDENCE_REQUIRED` 오류가 반환된다.

## 불변식

- **done에 Evidence 필수**: done 전환 시 시나리오 증거(scenarios 배열 또는 summary)가 비어 있으면 거부된다.
- **증거 내 reference 비어있으면 안 됨**: scenarios 배열 각 항목의 evidence_ref는 공백 이외의 값이어야 한다.
- **유효 전이만 허용**: 위에 나열된 전이 쌍 외의 전이는 InvalidTransition 오류로 거부된다.
- **인간 직접 작성 금지(D3)**: 태스크는 LLM이 시나리오를 분해해 생성하는 런타임 산물이다.

## 구현 위치 (provenance)

상태 전이 머신과 Evidence 검증(`check_transition`, `TaskEvidence::validate_for_done`)은 `sdi-plugin/crates/core/src/task.rs`에 있다. DB 스키마는 마이그레이션 001(핵심 엔티티), 010(short_code를 플랜 범위로 재조정)에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: done→todo 역전이가 가능한지 — 현재 check_transition 목록에는 없으나 "재개방이 일부 허용된다"는 PRD 주석과 비교해 정확한 허용 전이 완전 확인 필요.
