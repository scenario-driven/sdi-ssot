---
id: concept.scenario
kind: Concept
title: Scenario(시나리오)
definition: "SDI의 가장 핵심적인 1등 시민(first-class entity). 코드가 무엇을 해야 하는가를 자연어 Given/When/Then 형식으로 선언한 원자적 진술. 검증의 기준점이며, 라운드마다 LLM이 이 기준에 코드를 대조해 Pass/Fail 판정을 내린다."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "시나리오는 반드시 하나의 플랜에 속한다." }
  - { to: concept.given-when-then, type: backed-by, note: "시나리오의 본문은 GWT 형식 세 필드로 구성된다." }
  - { to: concept.scenario-id, type: relates-to, note: "시나리오는 short_code(예: SC-1)와 시스템 ID(SCN-...)를 통해 식별된다." }
  - { to: concept.scenario-claim, type: relates-to, note: "D29 다중 세션 환경에서 시나리오는 파일 경로 글로브 목록을 claim해 다른 세션의 중복 편집을 차단한다." }
  - { to: concept.round, type: relates-to, note: "라운드가 활성화될 때 시나리오들이 태스크로 분해되거나 회귀 검증 대상이 된다." }
  - { to: concept.task, type: relates-to, note: "분해된 태스크는 parent_scenario_ids로 자신이 어느 시나리오에서 비롯되었는지 가리킨다." }
  - { to: concept.disruption-review, type: relates-to, note: "새 시나리오/요건/결정이 기존 시나리오에 영향을 주면 DisruptionReview가 열린다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "시나리오는 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: domain.scenario-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/scenario.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/007_v05_pattern_enforcement.sql
  - sdi-plugin/crates/db/src/migrations/012_scenario_retire.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

시나리오는 SDI가 TDD·BDD로부터 계승한 "의도를 1등 시민으로" 원칙의 구현체다. "코드가 현재 무엇을 하는가"가 아니라 "코드가 무엇을 해야 하는가"를 한 덩어리로 선언하고, 라운드마다 LLM이 이 선언을 기준으로 코드를 판정한다.

시나리오는 항상 하나의 플랜에 속하며, 플랜 안에서 고유한 짧은 코드(short_code, 예: SC-1)를 갖는다. 같은 short_code를 다른 플랜에서 재사용할 수 있으나 동일 플랜 안에서는 중복 불가다. 시스템이 부여한 고유 식별자(SCN-...)는 항구적이며 재사용하지 않는다.

시나리오의 생애주기는 두 단계다.

- **draft**: 아직 승인되지 않은 초안. 이 상태에서는 플랜의 approve 조건에 산입되지 않는다.
- **confirmed**: 확정 완료. 플랜 approve 게이트(D8)는 confirmed 시나리오가 1개 이상인지를 검사한다.

시나리오는 **은퇴(retire)** 할 수 있다. 은퇴는 삭제가 아니라 "이 시나리오는 더 이상 검증 대상이 아님"을 표시하는 것이며, 이력은 보존된다. 은퇴는 취소해 복원할 수 있고, 은퇴·복원은 status(draft/confirmed)와 독립적으로 동작한다.

시나리오들은 `depends_on` 필드로 DAG(유향 비순환 그래프)를 형성할 수 있다. 한 시나리오가 다른 시나리오의 선행 결과에 의존하는 경우 사용하며, 자기 자신을 가리키는 즉각적 순환은 도메인 레벨에서 차단된다.

## 엔티티 (DB)

시나리오 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜의 참조, 최초 생성된 라운드 참조(optional).
- **본문**: Given, When, Then 세 필드. 각각 비어 있으면 안 된다(D5 GWT 엄격 규칙).
- **분류**: 태그 목록, 상태(draft/confirmed), 은퇴 시각(Some이면 은퇴됨).
- **DAG**: 선행 시나리오 short_code 목록(depends_on), 이 시나리오를 구현할 에이전트(produced_by), 검증할 에이전트(verified_by).
- **리소스 클레임(D29)**: 이 시나리오가 작업 중에 독점적으로 사용하겠다고 신고한 파일 경로 글로브 목록(JSON), 현재 클레임 상태(none/requested/active/released).
- **패턴 출처(D23)**: 이 시나리오가 어떤 협업 패턴 아래 만들어졌는지의 참조.
- **타임스탬프**: 생성 시각, 마지막 수정 시각.

## API 표면

시나리오를 다루는 경로는 두 가지다.

- **작성·관리**: `/sdi scenario` 명령(/scenario 슬래시 커맨드 포함)으로 시나리오를 추가·수정·은퇴·복원한다. 추가 시 GWT 필드가 모두 비어있지 않아야 한다.
- **라운드 연결**: 라운드 활성화 시 확정된 시나리오들이 태스크로 분해되거나 strict-regression 모드에서 이전 라운드의 시나리오들이 회귀 검증 대상으로 재도입된다.

## 불변식

- **GWT 완전성(D5)**: Given, When, Then 각각이 비어 있으면 시나리오를 저장할 수 없다.
- **short_code 유일성**: 동일 플랜 안에서 short_code는 중복될 수 없다.
- **자기 의존 금지**: depends_on에 자기 자신의 short_code를 포함할 수 없다.
- **리소스 클레임 형식(D29)**: claimed_resources_json은 JSON 배열이어야 하며, 각 항목은 비어 있지 않고 순수 와일드카드(`**`, `*`)만으로 된 글로브는 허용하지 않는다 — 최소한 하나의 리터럴 문자를 포함해야 한다.
- **플랜 approve 게이트(D8)**: confirmed 상태이고 은퇴되지 않은 시나리오가 1개 이상이어야 플랜을 approve할 수 있다.

## 구현 위치 (provenance)

시나리오 도메인 규칙(GWT 검증, depends_on 자기참조 차단, 리소스 클레임 파싱)은 `sdi-plugin/crates/core/src/scenario.rs`에 있다. DB 스키마는 마이그레이션 001(핵심 엔티티), 007(D29 클레임 컬럼), 012(은퇴 컬럼)에서 단계적으로 확장되었다.

## 미확정 (OPEN)

- [ ] OPEN: `origin_round_id`가 NULL 허용이지만 실제로 언제 채워지고 언제 비는지 — 마이그레이션된 행의 경우 NULL이 허용된다고만 확인됨.
- [ ] OPEN: 은퇴(retire)된 시나리오의 strict-regression 라운드 캐리오버 규칙 — 은퇴 시나리오가 회귀 대상에서 제외됨은 코드 주석으로 확인되나, approved_count 게이트와의 정확한 상호작용 검증 필요.
