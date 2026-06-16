---
id: invariant.one-active-plan
kind: Invariant
title: 한 프로젝트에 active 상태의 Plan은 하나뿐이다
definition: "한 Project 안에서 lifecycle이 active인 Plan은 동시에 하나만 존재할 수 있다. 새 Plan을 active로 전환하려면 기존 active Plan이 completed이어야 한다."
governs:
  - concept.plan
  - domain.planning
  - domain.project-management
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/plan.rs"
  - "sdi-plugin/crates/core/src/plan.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 Plan은 승인된 시나리오 집합과 그 아래 생성된 Round·Task의 컨테이너다. 한 프로젝트에 두 개의 active Plan이 동시에 존재하면 어떤 시나리오가 현재 목표이고 어떤 Round가 진행 중인지 구분할 수 없다. 훅 레이어의 PreToolUse 게이트도 "현재 활성 Plan"을 컨텍스트로 주입하므로, 복수의 active Plan은 어떤 Plan에 대한 작업인지 모호하게 만든다.

Plan의 lifecycle 전이: `draft → active → completed`. `draft` 상태의 Plan은 여러 개 동시에 존재할 수 있지만, `active`는 프로젝트 당 하나로 제한된다. Plan approve 게이트(`invariant.plan-active-for-task` 참조)는 이 전이 시점에 시나리오 최소 1개 + GWT 유효성 검사를 통과해야 한다(D8).

## 깨지면 무슨 일이 일어나나

두 Plan이 동시에 active가 되면 `/round start` 가 어느 Plan의 라운드를 시작하는지 불명확해진다. `UserPromptSubmit` 훅이 주입하는 활성 Plan/Round 컨텍스트가 두 Plan 중 어느 것인지 모호해져 LLM이 잘못된 시나리오 집합을 기반으로 Task를 분해할 수 있다. 회귀 검증에서 어떤 Plan의 시나리오가 passing이어야 하는지도 불분명해진다. Disruption 검사도 어느 Plan의 시나리오가 영향을 받는지 판단할 수 없다.

## 코드에서 어떻게 강제되나

데몬의 Plan approve 엔드포인트(`crates/daemon/src/router/plan.rs`)는 `draft → active` 전환 시 해당 프로젝트에 이미 active인 Plan이 존재하는지 확인한다. 존재하면 `ACTIVE_PLAN_EXISTS` 오류를 반환한다. 이 단일 게이트가 복수 active Plan의 생성을 원천 차단한다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(Plan active 단일성 강제 정책의 결정 엔트리) 연결 필요
