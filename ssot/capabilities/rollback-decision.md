---
id: capability.rollback-decision
kind: Capability
title: 결정 롤백 실행
purpose: "적용된 결정을 되돌려야 할 때 reversal_plan에 정의된 롤백 절차를 실행하고, 그 결과를 새 consensus 결정으로 추가 기록한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/decide.md
relatesTo:
  - { to: concept.decision, type: mutates, note: "롤백 결과는 reversal_of 필드를 가진 새 consensus 결정으로 추가된다(추가 전용 보존)" }
  - { to: concept.autonomy-policy, type: reads, note: "L5 잠금 해제 조건 중 하나가 reversal_plan NOT NULL + blast_radius_score ≤ l5_threshold(D28)" }
  - { to: concept.circuit-breaker, type: relates-to, note: "롤백이 필요한 상황은 circuit breaker 발동과 연관될 수 있다" }
  - { to: concept.round, type: relates-to, note: "롤백은 이미 적용된 결정 이후의 라운드 결과에도 영향을 미칠 수 있다" }
impacts:
  - concept.decision
  - concept.autonomy-policy
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

잘못된 결정이 이미 적용된 뒤 문제가 발견됐을 때 롤백을 실행한다. SDI에서 롤백은 "결정을 지운다"가 아니다. 롤백 행위 자체를 새 의사결정으로 추가 기록하는 방식으로, 기존 결정은 데이터베이스에 그대로 남고 `reversal_of` 링크로 연결된 새 결정이 생긴다. 이 추가 전용 원칙이 역사를 온전히 보존한다.

롤백 계획(reversal_plan)은 결정을 만들 때 미리 작성해 두어야 한다(D28). reversal_plan이 있고 blast_radius_score가 임계값 이하일 때만 L5 자동 적용이 허용된다—되돌릴 수 없는 결정은 자동화에서 제외된다. reversal-runner 전문 에이전트가 실제 롤백 절차(마이그레이션 SQL, git revert, 파일 스냅샷 복원, 보상 액션)를 실행한다.

## 행위

- 롤백 대상 결정 ID를 확인한다. `reversal_plan` 필드가 채워져 있어야 진행 가능.
- `sdi decision rollback <DEC-ID>` (또는 데몬의 `/decisions/<id>/rollback` 엔드포인트)를 호출해 reversal-runner를 트리거한다.
- reversal-runner가 `reversal_plan`의 종류(migration_sql / git_revert / fs_snapshot / compensating_action)에 따라 실행한다.
- 실행 결과를 `kind=consensus, reversal_of=<원본 ID>` 인 새 결정으로 추가한다.
- blast_radius_score > l5_threshold 이면 L5에서도 사람 확인 후 실행.

## 시스템 흐름

문제 발견 → 롤백 대상 결정 확인 → reversal_plan 유효성 검사 → `sdi decision rollback <DEC-ID>` → reversal-runner 에이전트 실행 → 롤백 액션 수행 → 결과를 `reversal_of=<DEC-ID>` 인 새 결정으로 추가 기록 → 원본 결정은 그대로 보존, 체인으로 연결.

## 어디에 구현되어 있나

롤백 절차는 `decide` 스킬(`plugin/commands/decide.md`)과 reversal-runner 전문 에이전트가 담당한다. 데몬 `/decisions/<id>/rollback` HTTP 엔드포인트가 트리거를 수신하고, reversal-runner가 실제 롤백 액션을 실행한다.

## 미확정 (OPEN)

- [ ] OPEN: reversal_plan 의 네 가지 종류(migration_sql / git_revert / fs_snapshot / compensating_action) 각각에 대한 reversal-runner의 구체 실행 로직이 구현됐는지 코드 확인 필요.
