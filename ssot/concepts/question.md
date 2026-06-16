---
id: concept.question
kind: Concept
title: Question(질문)
definition: "플랜에 달리는 단일-답변 Q&A 항목. open → answered 상태 기계를 따르며, 답변자(answered_by)와 답변 내용(answer)이 기록된다. 플랜 실행 중 불명확한 사항을 기록하고 해소 내역을 추적하는 경량 협업 도구."
relatesTo:
  - { to: concept.plan, type: belongs-to, note: "Question은 플랜에 속한다." }
  - { to: domain.collaboration-patterns, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/collab.rs
  - sdi-plugin/crates/db/src/migrations/003_collab.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Question(질문)은 플랜 실행 중 답변이 필요한 사항을 기록하고 추적하는 경량 Q&A 항목이다. 복잡한 의사결정을 위해서는 Decision 엔티티(M3 협상 포함)를 사용하고, 단순한 "이것이 어떻게 동작하는가?" 류의 질문에는 Question을 사용한다.

상태는 두 가지다.

- **open**: 아직 답변 없음.
- **answered**: 답변 완료. answer, answered_by, answered_at이 채워진다.

Question은 댓글(Comment)보다 구조화된 형태로, 반드시 답변이 있어야 해소되며 해소 주체(answered_by)를 명시적으로 기록한다.

## 엔티티 (DB)

Question 한 건은 다음 정보를 보존한다.

- **소속**: plan_id.
- **내용**: asker(질문자), body(질문 내용).
- **답변**: answer(답변 내용), answered_by(답변자), answered_at.
- **상태**: open/answered.
- **타임스탬프**: 생성, 수정 시각.

## API 표면

질문 생성·답변은 CLI 또는 대시보드에서 처리한다.

## 불변식

- **answered_by 필수(answered 시)**: answered 상태로 전환 시 answered_by와 answer가 있어야 한다.

## 구현 위치 (provenance)

`Question` 구조체는 `sdi-plugin/crates/core/src/collab.rs`에 있다. DB 스키마는 마이그레이션 003에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: answered → open 역전이(질문 재개방)가 허용되는지 확인 필요.
