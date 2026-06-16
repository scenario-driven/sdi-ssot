---
id: screen.timeline
kind: Screen
title: Timeline 뷰 (프로젝트 활동 피드)
purpose: "소로 빌더가 프로젝트의 모든 이벤트(플랜 생성·승인·완료, 라운드 생성·활성화·완료, 태스크 생성·증거 기록, 시나리오 추가, 결정 기록)를 시간 역순으로 파악한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/TimelineView.tsx
consumesApi: []
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜의 생성·승인·완료 시각을 타임라인에 표시한다" }
  - { to: concept.round, type: reads, note: "라운드 생성·활성화·완료 시각을 표시한다" }
  - { to: concept.task, type: reads, note: "태스크 생성 시각과 증거 기록 시각을 표시한다" }
  - { to: concept.scenario, type: reads, note: "시나리오 추가 시각을 표시한다" }
  - { to: concept.decision, type: reads, note: "결정 기록 또는 superseded 전환 시각을 표시한다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

프로젝트에서 일어난 모든 주요 사건을 시간 역순 피드로 보여주는 활동 기록 뷰다. 각 항목은 발생 시각, 이벤트 종류(plan/round/task/scenario/decision), short_code, 설명으로 구성된다. "언제 무엇이 생성되었고, 언제 승인·완료·증거 기록이 이루어졌는가"를 추적하는 감사 피드 역할을 한다.

## UI 요소 / 입력 필드

읽기 전용 피드다. 필터·검색 기능은 없다.

- **타임라인 피드**: 세로 선(vertical timeline) 위에 항목이 최신순으로 나열된다. 각 항목은 색상 원형 마커(플랜=파란색, 라운드=주황색, 태스크=초록색, 시나리오=보라색, 결정=회색), 발생 시각, short_code, 이벤트 레이블("plan created", "round activated", "task evidence recorded" 등), 보충 설명으로 구성된다.

## 표시 데이터 / 호출 API

화면 진입 시 프로젝트의 모든 플랜을 조회한다. 각 플랜마다 라운드·시나리오·결정 목록을 동시에 조회하고, 각 라운드마다 태스크 목록을 추가로 조회한다. 이 모든 엔티티의 시각 정보(생성·승인·완료·활성화·증거 기록)를 하나의 목록으로 합쳐 최신순으로 정렬한 뒤 화면에 표시한다.

## 상태 / 엣지케이스

- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **활동 없음**: "No activity yet." 안내를 표시한다.
- **시각 포맷**: 브라우저의 `toLocaleString()`으로 사용자 로케일에 맞게 표시된다.
- **부분 조회 실패**: 각 플랜의 라운드·시나리오·결정, 각 라운드의 태스크 조회가 실패하면 해당 항목만 조용히 빈 배열로 처리하고 나머지는 계속 표시한다.

## 미확정 (OPEN)
- [ ] OPEN: 데이터 양이 많을 때 페이지네이션·무한 스크롤 적용 여부 확인 필요.
