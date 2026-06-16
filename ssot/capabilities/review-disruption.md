---
id: capability.review-disruption
kind: Capability
title: 중단 리뷰 처리(Disruption Review)
purpose: "진행 중인 라운드에 영향을 주는 시나리오 변경이 발생했을 때 사람이 검토하고 수용 또는 거부해 라운드 활성화 차단을 해소한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-round/SKILL.md
relatesTo:
  - { to: concept.disruption-review, type: mutates, note: "리뷰를 승인 또는 거부해 DISRUPTION_PENDING 상태를 해소한다" }
  - { to: concept.scenario, type: reads, note: "기존 시나리오에 확정된 변경이 생겼을 때 중단 리뷰가 트리거된다" }
  - { to: concept.round, type: reads, note: "DISRUPTION_PENDING 인 플랜은 라운드 활성화가 차단된다" }
  - { to: concept.plan, type: reads, note: "플랜 단위로 중단 리뷰 상태가 관리된다" }
impacts:
  - concept.disruption-review
  - concept.round
  - concept.scenario
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

기존 confirmed 시나리오의 내용이 바뀌면—확정된 행동 명세가 변경된다는 것은 그 시나리오를 기반으로 진행 중이던 작업에 영향이 생긴다는 의미—시스템이 자동으로 중단 리뷰를 열고 라운드 활성화를 차단한다. 사람이 그 변경이 의도된 것인지 확인하고 수용 또는 거부한다.

수용하면(approve) 시나리오 변경이 확정되고 라운드가 활성화될 수 있다. 거부하면(reject) 변경이 취소되고 시나리오가 이전 상태로 돌아간다. 이 강제 검토가 "실수로 중요한 시나리오를 바꿨는데 모르고 라운드가 돌아갔다"는 상황을 막는다.

## 행위

- 시나리오 변경 시 데몬이 자동으로 중단 리뷰를 생성하고 플랜에 DISRUPTION_PENDING 을 붙인다.
- `sdi round activate` 시 DISRUPTION_PENDING 이면 차단된다.
- `sdi disruption resolve <REVIEW-ID> --approve` 로 변경을 수용한다.
- `sdi disruption resolve <REVIEW-ID> --reject` 로 변경을 거부한다.
- 해소 후 `sdi round activate` 를 재시도한다.

## 시스템 흐름

confirmed 시나리오 내용 변경 → 데몬이 중단 리뷰 레코드 생성, 플랜에 DISRUPTION_PENDING 표시 → `sdi round activate` 시 DISRUPTION_PENDING 이면 HTTP 409 반환 → 사람이 리뷰 확인 → `sdi disruption resolve --approve | --reject` → DISRUPTION_PENDING 해제 → `sdi round activate` 재시도 성공.

## 어디에 구현되어 있나

중단 리뷰 절차는 `sdi-round` 스킬(`plugin/skills/sdi-round/SKILL.md`)의 disruption review gate 섹션에 정의된다. 데몬이 시나리오 변경 감지, 리뷰 레코드 생성, DISRUPTION_PENDING 플래그 관리, resolve 처리를 수행한다.

## 미확정 (OPEN)

- [ ] OPEN: `needs-review` 기본 정책 외에 `auto` 정책 선택 시 중단이 사람 확인 없이 자동 수용되는지, 아니면 여전히 confirm 단계가 있는지 실제 동작 확인 필요.
