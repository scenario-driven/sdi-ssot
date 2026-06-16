---
id: concept.timeline-event
kind: Concept
title: Timeline Event / Activity(타임라인 이벤트)
definition: "프로젝트의 append-only 감사·타임라인 피드. 플랜 승인, 태스크 완료, 라운드 종료, 결정 생성 등 주요 상태 변화가 Activity로 기록된다. SSE(Server-Sent Events) 실시간 이벤트와 구별되며 Activity는 인간이 읽는 요약을 담는다."
relatesTo:
  - { to: concept.project, type: belongs-to, note: "Activity는 프로젝트에 속한다." }
  - { to: concept.plan, type: relates-to, note: "plan.approved 등 플랜 이벤트가 Activity로 기록된다." }
  - { to: concept.task, type: relates-to, note: "task.completed 등 태스크 이벤트가 Activity로 기록된다." }
  - { to: domain.governance-audit, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/collab.rs
  - sdi-plugin/crates/db/src/migrations/003_collab.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Timeline Event(타임라인 이벤트)는 SDI 코드에서 `Activity`로 구현된 append-only 감사 피드다. 시스템에서 중요한 일이 발생할 때마다(플랜 승인, 태스크 완료, 라운드 시작, 결정 생성 등) Activity 행이 자동으로 생성된다.

Activity와 SSE 이벤트(Server-Sent Events, 웹소켓 방식의 실시간 알림)는 다르다.

- **SSE 이벤트**: 실시간 푸시. 클라이언트(대시보드, CLI)가 구독해 실시간으로 상태 변화를 받는 와이어 레벨 이벤트.
- **Activity**: 영구 저장되는 감사·타임라인 기록. 인간이 읽는 요약 텍스트를 담는다.

Activity의 `kind` 필드는 `plan.approved`, `task.completed`, `round.completed`, `decision.created` 등 점 구분 형태로 이벤트 종류를 표현한다. 새로운 종류의 이벤트가 추가될 때 마이그레이션이 필요 없도록 자유 형식 문자열을 사용한다.

`payload` 필드에는 이벤트별 구조화된 추가 정보를 JSON으로 담는다.

## 엔티티 (DB)

Activity 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 프로젝트 참조, 선택적으로 관련 엔티티 참조(entity_id).
- **행위자**: actor(에이전트 이름 또는 사용자).
- **내용**: kind(자유 형식 이벤트 종류), summary(인간이 읽는 요약), payload(JSON 추가 정보).
- **타임스탬프**: created_at.

## API 표면

Activity는 daemon이 자동으로 기록하며 사용자가 직접 생성하지 않는다. 대시보드의 타임라인 뷰나 CLI에서 조회한다.

## 불변식

- **append-only**: Activity 행은 수정하거나 삭제하지 않는다.

## 구현 위치 (provenance)

`Activity` 구조체는 `sdi-plugin/crates/core/src/collab.rs`에 있다. DB 스키마는 마이그레이션 003에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: Activity 자동 생성이 daemon의 어느 레이어(route, repo, middleware)에서 이루어지는지 확인 필요.
