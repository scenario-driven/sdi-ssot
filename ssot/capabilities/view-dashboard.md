---
id: capability.view-dashboard
kind: Capability
title: 웹 대시보드 조회
purpose: "브라우저에서 현재 플랜의 시나리오 상태, 태스크 칸반, 패턴 현황, 라운드 타임라인, 지식 문서를 시각적으로 확인한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/web/src/views/SummaryView.tsx
  - sdi-plugin/plugin/web/src/views/BoardView.tsx
  - sdi-plugin/plugin/web/src/views/PatternsView.tsx
  - sdi-plugin/plugin/web/src/views/TimelineView.tsx
  - sdi-plugin/plugin/web/src/views/WikiView.tsx
  - sdi-plugin/plugin/web/src/views/BacklogView.tsx
  - sdi-plugin/plugin/web/src/views/NextView.tsx
relatesTo:
  - { to: concept.plan, type: reads, note: "활성 플랜 상태와 시나리오 목록을 SummaryView에서 표시한다" }
  - { to: concept.task, type: reads, note: "BoardView가 라운드 내 태스크를 칸반 방식으로 표시한다" }
  - { to: concept.collaboration-pattern, type: reads, note: "PatternsView가 활성·종결 패턴 현황을 표시한다" }
  - { to: concept.round, type: reads, note: "TimelineView가 라운드 이력과 판정 추이를 표시한다" }
  - { to: concept.knowledge-entry, type: reads, note: "WikiView가 플랜의 마크다운 지식 문서를 탐색·렌더한다" }
impacts:
  - concept.plan
  - concept.task
  - concept.round
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

`sdid`(데몬)가 실행 중이면 브라우저에서 SDI 대시보드 SPA에 접속해 현재 상황을 한눈에 파악한다. 슬래시 명령 없이 클릭으로 탐색하는 읽기 중심 인터페이스다. 대시보드는 여섯 가지 화면으로 구성된다.

**SummaryView**: 활성 플랜의 시나리오 목록과 확정 상태를 한눈에 본다. **BoardView**: 라운드 내 태스크를 칸반 컬럼(todo / in_progress / blocked / done)으로 표시한다. **PatternsView**: 현재 활성 협업 패턴과 라이프사이클 상태를 본다. **TimelineView**: 라운드 이력과 시나리오별 판정 추이를 시간 순으로 본다. **WikiView**: 플랜에 등록된 마크다운 지식 문서를 트리로 탐색하고 렌더링해 읽는다. **BacklogView / NextView**: 미착수 태스크와 다음 우선 작업을 확인한다.

## 행위

- 브라우저에서 `sdid`가 서빙하는 SPA에 접속한다.
- 상단 네비게이션으로 여섯 뷰를 전환한다.
- 각 뷰에서 항목을 선택해 상세를 펼친다.
- WikiView에서 지식 문서를 마크다운으로 읽는다.

## 시스템 흐름

`sdid` 데몬 시작 → tower-http ServeDir 로 `plugin/web/dist` SPA 파일 서빙 → 브라우저 접속 → SPA가 데몬 HTTP API를 폴링·조회해 데이터 표시 → 사용자가 뷰 전환 및 항목 탐색. 데몬 없이는 SPA가 데이터를 가져오지 못한다.

## 어디에 구현되어 있나

SPA는 Vite + React 19 + Tailwind 4 로 구현되며 `plugin/web/`에 위치한다. 여섯 개 뷰 컴포넌트가 `plugin/web/src/views/` 에 있다. 빌드 산출물(`plugin/web/dist/`)을 데몬이 `tower-http ServeDir` 로 직접 서빙한다. 별도 웹서버가 필요하지 않다.

## 미확정 (OPEN)

- [ ] OPEN: SPA에서 읽기 이외 쓰기 동작(플랜 승인, 태스크 상태 변경 등)을 지원하는지 현재 구현 상태 확인 필요.
