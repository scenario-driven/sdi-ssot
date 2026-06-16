---
id: concept.disruption-review
kind: Concept
title: DisruptionReview(혼란 검토)
definition: "LLM이 새 시나리오/요건/결정을 추가하면서 기존 시나리오들이 영향을 받는다고 판단할 때 열리는 검토 대기열 항목. 이 항목이 pending 상태로 있는 한 해당 플랜의 라운드는 활성화할 수 없다. 인간이 각 검토를 retire/edit/keep으로 해소해야 한다."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "DisruptionReview는 플랜에 속한다." }
  - { to: concept.scenario, type: relates-to, note: "impacted_scenario_ids는 영향받은 기존 시나리오들을 열거한다." }
  - { to: concept.round, type: relates-to, note: "pending DisruptionReview가 있으면 라운드를 active로 전환할 수 없다." }
  - { to: domain.disruption-review, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/disruption.rs
  - sdi-plugin/crates/db/src/migrations/002_disruption_reviews.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

DisruptionReview(혼란 검토)는 SDI가 "기존 시나리오에 영향을 줄 수 있는 변경이 감지되었을 때 인간이 먼저 확인해야 한다"는 원칙(D9)을 구현하는 검토 대기열 항목이다.

LLM이 새로운 시나리오·요건·결정을 추가할 때, 그 변경이 기존 시나리오들의 의미나 유효성에 영향을 줄 수 있다고 판단하면 DisruptionReview를 생성한다. "영향이 있는지"는 LLM의 의미적 판단이며 daemon은 강제하지 않는다. daemon의 역할은 이 대기열을 보관하고, 미해소 항목이 있는 한 라운드 활성화를 차단하는 것이다.

검토 해소 방법(resolution)은 세 가지다.

- **retire**: 영향받은 시나리오들을 은퇴 처리.
- **edit**: 시나리오들을 수정해 영향을 흡수.
- **keep**: 영향이 있음을 인지하되 시나리오들을 그대로 유지.

인간이 resolution을 명시하지 않고 simply 검토를 acknowledged하는 방식으로 해소해도 된다. 중요한 것은 `status = resolved`가 되는 것이다.

DisruptionReview는 반드시 최소 1개의 `impacted_scenario_ids`를 가져야 한다. "영향이 없다"는 결론에는 DisruptionReview를 생성하지 않는다.

## 엔티티 (DB)

DisruptionReview 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜 참조.
- **원인**: source_kind(scenario/requirement/decision), source_id(원인이 된 새 엔티티의 ID).
- **대상**: impacted_scenario_ids(영향받은 기존 시나리오 ID 목록, 최소 1개).
- **상태**: pending/resolved.
- **해소 정보**: resolution(retire/edit/keep, optional), note(자유 서술), resolved_at.
- **타임스탬프**: created_at.

## API 표면

DisruptionReview는 LLM이 시나리오·요건·결정을 추가하는 write path에서 자동 생성될 수 있다. 사용자(인간)가 대기열을 조회하고 각 항목을 해소한다. 라운드 활성화 API는 내부적으로 pending 항목 존재 여부를 확인한다.

## 불변식

- **최소 1개 impacted 시나리오**: 영향 대상 없는 DisruptionReview는 생성할 수 없다.
- **라운드 게이트**: pending 항목이 있으면 해당 플랜의 라운드를 active로 전환할 수 없다.
- **D9 인간 확인 필수**: 라운드 `disruption_policy=auto`이더라도 이 게이트는 우회되지 않는다. auto는 LLM이 해소 제안을 자동으로 만드는 것을 허용하되 인간 확인은 여전히 필요하다.

## 구현 위치 (provenance)

DisruptionReview 도메인 규칙(`validate_create`)은 `sdi-plugin/crates/core/src/disruption.rs`에 있다. DB 스키마는 마이그레이션 002에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: LLM이 DisruptionReview를 생성하는 write path가 daemon에서 어떻게 연결되는지 — 시나리오 추가 API에서 자동으로 생성하는지, 별도 API를 호출해야 하는지 확인 필요.
