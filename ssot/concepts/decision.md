---
id: concept.decision
kind: Concept
title: Decision(결정)
definition: "append-only ADR(아키텍처 결정 기록). 한 번 기록한 결정 행은 수정하지 않으며, 나중에 다른 결정이 supersedes_id로 체인해 교체한다. D20의 M3 4단계 협상 흐름(proposal → critique → consensus | dissensus)을 구현하는 엔티티."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "결정은 반드시 하나의 플랜에 속한다." }
  - { to: concept.consensus, type: relates-to, note: "M3 흐름의 최종 단계가 consensus이며, 이 결정이 accepted로 전환된다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "결정 종류(architecture/schema/naming-canonical)에 따라 강제 L4 규칙이 적용된다(D17)." }
  - { to: concept.circuit-breaker, type: relates-to, note: "D18 서킷 브레이커가 발동되면 모든 autonomy mode가 L3으로 강등되어 결정 자동 적용이 차단된다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "결정은 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: domain.decision-negotiation, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/decision.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/013_decision_supersede_when.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Decision(결정)은 SDI의 append-only 의사결정 기록이다. 한 번 기록된 결정 행은 절대 수정하지 않는다(D12). 나중에 같은 주제에 대해 다른 결정이 필요하면, 새 결정이 `supersedes_id`로 이전 결정을 가리키고 이전 결정의 status가 `superseded`로 전환된다. 이렇게 체인을 형성해 의사결정 히스토리를 보존한다.

결정의 상태는 세 가지다.

- **proposed**: 제안됨. 아직 수락되지 않음.
- **accepted**: 수락됨. 적용 중.
- **superseded**: 다른 결정에 의해 대체됨. 내용은 보존됨.

결정은 M3 4단계 협상 흐름(D20)을 구현하기 위해 `kind` 필드를 갖는다.

- **proposal**: 결정 흐름의 첫 번째 단계. `proposal_id`를 갖지 않는다.
- **critique**: 제안에 대한 비평. `proposal_id`로 해당 proposal을 가리킨다.
- **consensus**: 제안과 비평을 거쳐 도달한 합의. `proposal_id` 필수, 그리고 같은 proposal_id에 critique가 최소 1개 있어야 한다.
- **dissensus**: 합의 실패. `proposal_id` 필수. 항상 인간 게이트로 에스컬레이션된다.

**가역성(D28)**: accepted 상태의 consensus 결정은 `reversal_plan`과 `blast_radius_score`를 가져야 L5 자동 적용이 가능하다. reversal_plan은 네 가지 중 하나다 — SQL 역마이그레이션(`migration_sql`), git revert(`git_revert`), 파일 스냅샷 복원(`fs_snapshot`), 외부 보상 액션(`compensating_action`). blast_radius_score는 0~10 사이의 복구 비용 점수이며 기본값은 5다.

**잠정 결정(provisional)**: `supersede_when` 필드가 채워진 결정은 잠정 결정이다. accepted 상태이며 현재 적용 중이지만, 지정된 조건이 충족되면 다시 검토한다는 표시다.

## 엔티티 (DB)

결정 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜 참조.
- **식별**: 플랜 내 고유 short_code, 제목.
- **본문**: append-only 자유 서술 본문(수정 불가).
- **상태**: proposed/accepted/superseded.
- **체인**: supersedes_id(이 결정이 대체하는 이전 결정), reversal_of(이 결정이 롤백하는 결정).
- **협상 메타**: kind(proposal/critique/consensus/dissensus), proposal_id(proposal이 아닌 경우), agent_name(어떤 에이전트가 발행했는지), escalated_at.
- **가역성(D28)**: reversal_plan(JSON), blast_radius_score(0~10).
- **잠정**: supersede_when(조건 문자열).
- **패턴 출처(D23)**: produced_via_pattern_id.
- **타임스탬프**: created_at(수정 없음).

## API 표면

결정은 `/sdi decide` 명령(/decide 슬래시 커맨드 포함)으로 생성한다. 생성 시 M3 단계 전이 규칙을 검사한다(consensus는 critique 없이 생성 불가). 롤백은 `/decisions/:id/rollback` 경로로 reversal-runner specialist가 처리한다.

## 불변식

- **append-only(D12)**: 결정 행은 수정 불가. 교체는 supersedes_id 체인으로.
- **M3 순서(D20)**: proposal → critique(하나 이상) → consensus | dissensus. critique 없이 consensus 생성 불가.
- **blast_radius_score 범위**: 0 이상 10 이하.
- **L5 unlock 조건(D28)**: 패턴 형태 유효 + reversal_plan NOT NULL + blast_radius_score ≤ l5_threshold.
- **architecture/schema/naming-canonical 강제 L4(D17)**: 이 종류의 결정은 AutonomyMode L5가 허용되지 않는다.

## 구현 위치 (provenance)

결정 도메인 규칙(M3 단계 전이 검증, blast_radius_score 범위, reversal_plan 파싱)은 `sdi-plugin/crates/core/src/decision.rs`에 있다. reversal_plan JSON 파싱은 `crates/core/src/pattern.rs`의 `validate_reversal_plan_json`에 있다. supersede_when 컬럼은 마이그레이션 013에서 추가되었다.

## 미확정 (OPEN)

- [ ] OPEN: dissensus 결정 발생 시 에스컬레이션 흐름(escalated_at 설정 + 인간 게이트)이 daemon 레벨에서 어떻게 구현되는지 확인 필요.
