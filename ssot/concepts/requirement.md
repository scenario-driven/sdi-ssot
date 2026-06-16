---
id: concept.requirement
kind: Concept
title: Requirement(요건)
definition: "플랜에 속한 스냅샷 전용 요건 항목. 제품이 충족해야 할 조건·제약을 선언하며, 본문은 현재 상태만 담고 변경 이력은 본문에 남기지 않는다. 의사결정(Decision)의 append-only 로그가 변경 근거를 보존한다."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "요건은 반드시 하나의 플랜에 속한다." }
  - { to: concept.decision, type: relates-to, note: "요건 본문이 바뀐 경우 그 근거는 Decision에 append-only로 기록된다. 요건 자체는 snapshot-only(D12)." }
  - { to: concept.task, type: relates-to, note: "LLM이 요건을 분해할 때 태스크의 parent_requirement_ids가 이 요건을 가리킨다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "요건은 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: concept.disruption-review, type: relates-to, note: "새 요건이 기존 시나리오에 영향을 주면 DisruptionReview가 열린다." }
  - { to: domain.requirements, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/requirement.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Requirement(요건)는 플랜이 달성해야 할 조건이나 제약을 서술하는 항목이다. 시나리오(GWT 형식)와 달리 요건은 자유 서술 마크다운 본문을 갖는다. 두 개념은 역할이 다르다. 시나리오는 "코드가 어떻게 동작해야 하는가"를 테스트 가능한 Given/When/Then으로 표현하고, 요건은 "이 플랜이 충족해야 할 조건이 무엇인가"를 명시한다.

요건은 **스냅샷 전용(D12)** 이다. 본문을 수정하면 이전 본문은 사라지고 새 본문이 그 자리를 대체한다. 변경 흔적(취소선, "이전에는 A였지만 이제는 B", 버전 마커 등)을 본문에 남기는 것은 시스템이 감지해 거부한다. 변경의 맥락과 근거는 Decision(의사결정) 항목에 append-only로 별도 기록한다.

요건은 플랜 안에서 short_code로 식별된다. 어떤 요건이 "결정에 의해 재작성"된 경우 source 필드에 "decision-rewrite"가 기록되어 이 요건이 결정 흐름의 산물임을 표시한다.

## 엔티티 (DB)

요건 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 플랜의 참조.
- **식별**: 플랜 내 고유 short_code, 제목.
- **본문**: 마크다운 본문(스냅샷 전용). 변경 이력 패턴이 감지되면 저장이 거부된다.
- **출처**: 이 요건이 어떻게 플랜에 진입했는지("snapshot" 기본값, "decision-rewrite"는 결정이 본문을 재작성한 경우).
- **패턴 출처(D23)**: produced_via_pattern_id — 어떤 협업 패턴 아래 만들어졌는지.
- **타임스탬프**: 생성 시각, 마지막 수정 시각.

## API 표면

요건은 `/sdi req` 명령(/req 슬래시 커맨드 포함)으로 추가·수정·조회한다. 본문 수정 시 스냅샷 위반 감지기가 먼저 동작해 이력 흔적이 있으면 거부한다. 요건 목록은 해당 플랜을 조회할 때 함께 반환된다.

## 불변식

- **스냅샷 전용(D12)**: 본문에 다음 패턴이 있으면 저장이 거부된다 — 마크다운 취소선(`~~`), 인라인 버전 마커(`v1:`, `v2:`, `(was:`, `(원래:` 등), 영어 이력 문구(`previously`, `used to be`, `changelog:` 등), 한국어 이력 문구(`이전에는`, `원래는`, `기존 안`, `변경 이력` 등). 변경 근거는 Decision에 분리 보존한다.
- **short_code 유일성**: 동일 플랜 내에서 short_code는 중복될 수 없다.

## 구현 위치 (provenance)

요건 도메인 규칙(스냅샷 본문 위반 감지, 이력 패턴 목록)은 `sdi-plugin/crates/core/src/requirement.rs`에 있다. DB 스키마는 마이그레이션 001에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: source="decision-rewrite" 설정이 daemon write path에서 자동으로 이루어지는지, 호출자가 명시적으로 전달해야 하는지 확인 필요.
