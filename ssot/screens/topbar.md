---
id: screen.topbar
kind: Screen
title: 상단 탑바 (뷰 내비게이션 셸)
purpose: "뷰 탭 전환, 데몬 연결 상태 확인, 활성 CollaborationPattern 수 배지, 테마 전환, 커맨드 팔레트 열기를 하나의 고정 헤더 바에서 제공한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/components/shell/Topbar.tsx
  - sdi-plugin/plugin/web/src/components/shell/AppShell
consumesApi: []
relatesTo:
  - { to: concept.collaboration-pattern, type: reads, note: "활성 패턴 수와 direct 안티패턴 존재 여부를 배지로 표시한다" }
  - { to: screen.patterns, type: leads-to, note: "패턴 배지 클릭 시 Patterns 뷰로 이동한다" }
  - { to: screen.summary, type: leads-to }
  - { to: screen.next, type: leads-to }
  - { to: screen.board, type: leads-to }
  - { to: screen.backlog, type: leads-to }
  - { to: screen.timeline, type: leads-to }
  - { to: screen.wiki, type: leads-to }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

대시보드 상단 고정 바로, 사용자가 Summary·Next·Board·Backlog·Timeline·Wiki·Patterns 7개 뷰 중 하나를 탭 방식으로 전환하는 주된 내비게이션이다. 우측에는 활성 CollaborationPattern 수 배지, 데몬 연결 상태 표시, 테마 전환 버튼, 커맨드 팔레트 열기 버튼이 있다.

## UI 요소 / 입력 필드

- **뷰 탭 목록**: Summary / Next / Board / Backlog / Timeline / Wiki / Patterns 7개 탭 버튼. 현재 활성 뷰는 배경색으로 강조된다.
- **활성 패턴 배지**: 현재 프로젝트의 active 상태 CollaborationPattern 개수를 숫자로 표시한다. kind가 `direct`인 패턴이 하나라도 있으면 배지 위에 빨간 점(D23 안티패턴 경고)이 붙는다. 클릭하면 Patterns 뷰로 이동한다.
- **데몬 상태 표시**: 데몬이 응답 중이면 초록 점과 "daemon ok", 끊어지면 빨간 점과 "daemon down"을 표시한다. "daemon down" 상태일 때 클릭하면 페이지를 리로드해 재연결을 시도한다.
- **테마 전환 버튼**: system → dark → light 순으로 순환하며 테마를 바꾼다. 로컬 저장소에 기억된다.
- **커맨드 팔레트 열기(⌘K)**: 클릭 또는 Cmd+K / Ctrl+K 단축키로 전역 커맨드 팔레트를 연다.

## 표시 데이터 / 호출 API

탑바 자체는 데이터를 직접 조회하지 않는다. 앱 루트에서 계산된 `activePatternCount`와 `hasDirectAnti`, `daemonHealthy` 값을 props로 받아 표시한다. 데몬 헬스는 앱 루트에서 10초마다 폴링하고, 패턴 카운트는 SSE 구조 이벤트마다 갱신된다.

## 상태 / 엣지케이스

- **데몬 오프라인**: 빨간 점 + "daemon down" 표시, 클릭으로 재연결.
- **패턴 카운트 0**: 배지에 0이 표시되며 클릭 시 Patterns 뷰로 이동한다.
- **direct 안티패턴**: 배지 위 빨간 점이 표시되어 즉각 주의를 유도한다.

## 미확정 (OPEN)
- [ ] OPEN: 7개 뷰 탭이 좁은 화면 너비에서 어떻게 처리되는지(스크롤, 오버플로) 확인 필요.
