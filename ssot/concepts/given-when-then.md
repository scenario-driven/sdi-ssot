---
id: concept.given-when-then
kind: Concept
title: Given/When/Then (GWT)
definition: "시나리오의 본문 형식. Given(선행 상태), When(트리거/행위), Then(기대 결과) 세 필드가 모두 채워져야 하며 하나라도 비어 있으면 시나리오로 인정하지 않는다(D5 엄격 규칙). BDD(Behavior-Driven Development)의 'Gherkin' 형식의 SDI 후계."
relatesTo:
  - { to: concept.scenario, type: backed-by, note: "GWT는 시나리오 본문의 구조화 형식이다. 시나리오 없이 GWT만 따로 존재하지 않는다." }
  - { to: domain.scenario-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/scenario.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Given/When/Then(GWT)은 시나리오가 "검증 가능한 단언"이 되기 위한 세 부분이다.

- **Given(주어진 상태)**: 이 시나리오가 전제로 하는 선행 조건이나 맥락. 예: "사용자가 플랜 PLAN-A에 속한 확정 시나리오가 있을 때".
- **When(발생하는 일)**: 시나리오가 다루는 트리거 또는 행위. 예: "사용자가 `/sdi plan approve PLAN-A`를 실행하면".
- **Then(기대 결과)**: 위 행위 후 코드가 어떤 결과를 내야 하는지. 예: "플랜 상태가 active로 전환되고 confirmed 시나리오 count가 반환된다".

SDI가 D5 결정으로 "GWT 엄격 규칙"을 확립한 배경은, 자유 서술 시나리오(Given/When/Then 없이 한 문장으로만 쓴 것)가 검증 대상으로서 너무 모호해 LLM이 판정을 내리기 어렵기 때문이다. 세 필드가 명확히 채워질 때 LLM이 코드와 기계적으로 대조할 수 있다.

세 필드 각각은 공백만으로 이루어져서는 안 된다. 앞뒤 공백을 제거한 후에도 비어 있으면 GWT 검증이 실패한다.

## 엔티티 (DB)

GWT는 독립 테이블이 아니다. `scenarios` 테이블의 `given`, `when_clause`, `then_clause` 세 컬럼이 GWT를 구성한다. `when`은 Rust 예약어이므로 내부 필드명은 `when_clause`지만 직렬화·API에서는 `when`으로 노출된다.

## API 표면

시나리오 생성·수정 시 세 필드가 모두 전달되어야 한다. 어느 하나라도 비어 있으면 `GwtEmpty` 도메인 오류가 반환되어 시나리오가 저장되지 않는다.

## 불변식

- **세 필드 완전성(D5)**: Given, When, Then 중 하나라도 공백 제거 후 비어 있으면 저장 거부.
- **선택형 없음**: "자유 서술" 또는 "GWT 생략" 옵션은 존재하지 않는다. 모든 시나리오는 GWT 형식이다.

## 구현 위치 (provenance)

GWT 검증 로직(`validate_gwt`)은 `sdi-plugin/crates/core/src/scenario.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: 각 필드의 최대 길이 제한 — 현재 코드에서 최대 길이 검사가 보이지 않음. DB 레벨 또는 API 레벨에서 제한이 있는지 확인 필요.
