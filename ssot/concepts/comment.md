---
id: concept.comment
kind: Concept
title: Comment(댓글)
definition: "플랜·태스크·시나리오·라운드 중 정확히 하나에 닻을 내리는 사용자 댓글. 협업 도구의 스레드형 댓글과 유사하며 작성자(author)와 본문(body)을 담는다."
relatesTo:
  - { to: concept.plan, type: relates-to, note: "plan_id를 갖는 댓글은 플랜에 달린 것이다." }
  - { to: concept.task, type: relates-to, note: "task_id를 갖는 댓글은 태스크에 달린 것이다." }
  - { to: concept.scenario, type: relates-to, note: "scenario_id를 갖는 댓글은 시나리오에 달린 것이다." }
  - { to: concept.round, type: relates-to, note: "round_id를 갖는 댓글은 라운드에 달린 것이다." }
  - { to: concept.project, type: belongs-to, note: "댓글은 프로젝트에 속한다." }
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

Comment(댓글)는 플랜·태스크·시나리오·라운드 중 하나에 연결되는 사용자 메모다. 에이전트 간 통신에는 AgentNote를 사용하고, 사람이 작업에 메모나 맥락을 남길 때 Comment를 사용한다.

댓글은 반드시 정확히 하나의 닻(anchor)에 연결된다. plan_id, task_id, scenario_id, round_id 중 정확히 하나만 설정되어야 한다. 0개 또는 2개 이상이면 도메인 레벨에서 거부된다.

댓글은 수정 가능하다(updated_at이 있음).

## 엔티티 (DB)

댓글 한 건은 다음 정보를 보존한다.

- **소속**: project_id, 그리고 한 가지 닻(plan/task/scenario/round ID 중 정확히 하나).
- **내용**: author, body.
- **타임스탬프**: 생성, 수정 시각.

## API 표면

댓글은 CLI 또는 대시보드에서 생성·수정한다. 해당 엔티티 조회 시 댓글 목록이 포함될 수 있다.

## 불변식

- **정확한 하나의 닻**: plan/task/scenario/round 중 정확히 하나.

## 구현 위치 (provenance)

`Comment` 구조체와 `validate_anchor` 메서드는 `sdi-plugin/crates/core/src/collab.rs`에 있다. DB 스키마는 마이그레이션 003에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: 댓글 삭제 허용 여부 — AgentNote와 달리 Comment는 삭제 가능한지 확인 필요.
