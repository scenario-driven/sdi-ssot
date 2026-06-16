---
id: endpoint.disruption-reviews
kind: Endpoint
title: "POST /disruption-reviews"
definition: >
  요구사항, 결정, 또는 시나리오 변경이 기존 시나리오에 영향을 미칠 수 있다고 LLM이 판단했을 때,
  그 영향을 기록하기 위해 혼란 검토(disruption review)를 생성하는 엔드포인트.
  생성된 검토는 pending 상태로 시작하며, 미해결 검토가 있으면 라운드 활성화가 차단된다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/disruption.rs"
relatesTo:
  - to: concept.disruption-review
    type: mutates
  - to: concept.scenario
    type: reads
    note: "impacted_scenario_ids가 영향받는 시나리오 목록을 참조한다"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

LLM이 구현 작업 중 요구사항, 결정, 또는 시나리오의 변경이 기존에 통과된 시나리오에 영향을 줄 수 있다고 감지했을 때 그 사실을 기록하는 엔드포인트다. 생성된 혼란 검토는 `pending` 상태로 시작하며, 플랜 안에 미해결(pending) 검토가 하나라도 있으면 해당 플랜의 새 라운드 활성화가 차단된다(needs-review 게이트). 사람이 검토하고 `/disruption-reviews/:id/resolve`를 호출해야만 게이트가 해제된다.

검토가 생성될 때 `disruption.created` SSE 이벤트가 발행된다.

## 요청 / 응답

**POST /disruption-reviews**

요청 본문:

- `plan_id` (필수): 이 혼란 검토가 속하는 플랜 식별자.
- `source_kind` (필수): 변경을 일으킨 원천의 종류. 예를 들어 요구사항, 결정, 또는 시나리오.
- `source_id` (필수): 변경이 발생한 원천 엔티티의 식별자.
- `impacted_scenario_ids` (필수, 목록): 이 변경으로 영향받을 수 있는 시나리오 식별자 목록. 하나 이상이어야 한다.
- `note` (선택): 영향 내용이나 우려 사항을 자유 텍스트로 보충 기술.

응답: 생성된 혼란 검토 행. 상태는 `pending`.

## 권한 / 제약

- 생성된 검토는 `pending` 상태에서 자동으로 전환되지 않는다. 반드시 사람이 직접 `/disruption-reviews/:id/resolve`를 호출해야 해제된다.
- 미해결 검토가 있는 플랜에서는 새 라운드 활성화가 차단된다.
- `impacted_scenario_ids`는 빈 목록일 수 없다.
- `disruption.created` SSE 이벤트가 발행된다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/disruption.rs`에서 유추됨. needs-review 게이트와 LLM 영향 감지 패턴에서 참조.

## 미확정 (OPEN)

- `source_kind`의 허용 열거값 전체 목록 미확인.
- 동일한 `source_id`에 대해 중복 혼란 검토 생성을 허용하는지 방지하는지 미확인.
- `impacted_scenario_ids`에 존재하지 않는 시나리오 ID를 포함했을 때의 처리(오류 또는 무시) 미확인.
- SSE 이벤트 페이로드 구조 미확인.
