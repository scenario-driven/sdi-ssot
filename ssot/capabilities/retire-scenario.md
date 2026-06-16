---
id: capability.retire-scenario
kind: Capability
title: 시나리오 은퇴(Retire) 및 복원
purpose: "더 이상 유효하지 않거나 원자성을 잃은 시나리오를 이력을 보존하면서 검증 대상에서 제외하고, 필요 시 복원한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-scenario/SKILL.md
relatesTo:
  - { to: concept.scenario, type: mutates, note: "retired 상태로 전환하거나 다시 이전 상태로 복원한다" }
  - { to: concept.round, type: relates-to, note: "retired 시나리오는 라운드의 검증 대상과 회귀 이월 대상에서 제외된다" }
  - { to: concept.regression, type: relates-to, note: "은퇴한 시나리오는 strict-regression 이월에서 빠지므로 회귀 신호에 영향을 준다" }
  - { to: concept.scenario-id, type: relates-to, note: "한 번 발번된 ID는 은퇴 후에도 재사용하지 않아 라운드 간 추적성을 보존한다" }
impacts:
  - concept.scenario
  - concept.round
  - concept.regression
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

시나리오가 중복되거나, 여러 동작이 섞여 원자성을 잃었거나, 제품 방향이 바뀌어 더 이상 검증할 필요가 없어졌을 때 해당 시나리오를 "은퇴"시킨다. 은퇴는 삭제와 다르다. 시나리오 행 자체는 데이터베이스에 남아 과거 라운드의 판정(verdict)을 그대로 보존한다. 다음 라운드부터 검증 대상과 strict-regression 이월 목록에서 빠질 뿐이다.

비원자 시나리오를 분리할 때 표준 흐름은: 원본을 대체하는 여러 개의 새 시나리오를 확정(confirmed)하고, 그 후 원본을 은퇴시킨다. 원본 ID를 재사용하지 않음으로써 라운드 간 추적이 유지된다.

## 행위

- `sdi scenario retire <SCN-ID>` 로 검증 대상에서 제외한다.
- `sdi scenario unretire <SCN-ID>` 로 은퇴 이전 상태(draft 또는 confirmed)로 복원한다.
- 분리 시나리오 작성 → 원본 은퇴 순서를 따른다.
- 은퇴 ID는 영구 비워두고 재사용하지 않는다.

## 시스템 흐름

은퇴할 시나리오 식별 → `sdi scenario retire <SCN-ID>` 호출 → 데몬이 상태를 `retired` 로 변경 → 다음 `sdi round activate` 시 해당 시나리오가 needs-verification 목록에서 제외 → strict-regression 모드에서도 이월 대상에서 제외. 복원은 `unretire` 호출 하나로 이전 상태를 되살린다.

## 어디에 구현되어 있나

retire / unretire subcommand는 `sdi-scenario` 스킬(`plugin/skills/sdi-scenario/SKILL.md`)에 절차가 정의되어 있고, CLI(`sdi scenario retire / unretire`)로 실행된다. 데몬이 상태 전환과 라운드 검증 대상 목록 갱신을 처리한다.

## 미확정 (OPEN)

- [ ] OPEN: unretire 후 복원되는 정확한 상태(draft vs confirmed)가 데몬에서 어떻게 보존되는지 코드 확인 필요.
