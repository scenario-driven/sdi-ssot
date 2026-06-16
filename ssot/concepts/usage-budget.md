---
id: concept.usage-budget
kind: Concept
title: UsageBudget / UsageRecord(사용량 예산)
definition: "에이전트 실행별 토큰·도구 호출 수·비용을 기록하는 UsageRecord와, 플랜·태스크 범위로 집계하는 UsageRollup. 비용 추적과 예산 초과 사전 확인(preflight)에 사용된다."
relatesTo:
  - { to: concept.run, type: relates-to, note: "UsageRecord는 run_id로 특정 Run에 연결된다." }
  - { to: concept.task, type: relates-to, note: "UsageRecord는 task_id로 태스크에 연결되며 태스크별 비용 집계에 사용된다." }
  - { to: concept.plan, type: relates-to, note: "플랜 범위 집계(/plans/:id/usage)가 전체 플랜 비용을 보여준다." }
  - { to: concept.project, type: belongs-to, note: "UsageRecord는 project_id로 프로젝트에 속한다." }
  - { to: domain.governance-audit, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/usage.rs
  - sdi-plugin/crates/db/src/migrations/005_usage.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

UsageBudget(사용량 예산)은 두 가지 개념을 포함한다.

**UsageRecord**: 하나의 에이전트 실행 시도(Run)에서 사용된 자원을 기록한 행. 어떤 AI 모델을 사용했는지, 입력·출력 토큰 수, 캐시 읽기·쓰기 토큰, 도구 호출 횟수, 달러 비용 등을 담는다.

**UsageRollup**: 여러 UsageRecord를 특정 범위(플랜, 태스크)로 집계한 요약. records 수, 총 토큰, 총 비용 등을 합산한다.

`UsageTier`는 실행의 비용 등급이다. low/med(기본값)/high 세 단계가 있으며, 태스크의 tier와 대응한다. v3에서는 advisory(조언) 목적이며 v4에서 hard-enforce가 예정된 것으로 코드 주석에 명시되어 있다.

예산 초과 사전 확인은 `/tasks/:id/usage/preflight`로 태스크를 시작하기 전에 예상 비용이 허용 범위인지 미리 검사한다.

## 엔티티 (DB)

UsageRecord 한 건은 다음 정보를 보존한다.

- **소속**: project_id, 선택적으로 plan_id, task_id, run_id.
- **모델**: 사용된 AI 모델 이름.
- **등급**: tier(low/med/high).
- **수치**: input_tokens, output_tokens, cache_read, cache_write, tool_calls, cost_usd.
- **타임스탬프**: created_at.

## API 표면

에이전트는 실행 완료 후 UsageRecord를 daemon에 보고한다. 집계는 `/plans/:id/usage`와 `/tasks/:id/usage/preflight`로 조회한다.

## 불변식

- **tier 값**: low/med/high 세 값만 허용.

## 구현 위치 (provenance)

UsageRecord, UsageRollup, UsageTier는 `sdi-plugin/crates/core/src/usage.rs`에 있다. DB 스키마는 마이그레이션 005에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: v4에서 hard-enforce가 예정된 tier 강제 메커니즘의 구체적 설계 확인 필요.
- [ ] OPEN: preflight에서 "허용 범위"를 어떻게 정의하는지 — 플랜별 예산 상한 설정 API가 있는지 확인 필요.
