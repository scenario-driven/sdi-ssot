---
id: screen.sidebar
kind: Screen
title: 좌측 사이드바 (셸 네비게이션)
purpose: "소로 빌더가 프로젝트를 전환하고, 활성 플랜·라운드 컨텍스트를 한눈에 확인하며, 플랜 트리(Plan → Scenario/Requirement/Decision/Round/Task)를 탐색하는 상시 노출 패널이다."
servesPersona:
  - persona.solo-builder
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/components/Sidebar.tsx
  - sdi-plugin/plugin/web/src/components/shell/ProjectSwitcher.tsx
  - sdi-plugin/plugin/web/src/components/shell/BrandMark.tsx
  - sdi-plugin/plugin/web/src/components/PlanTree.tsx
consumesApi: []
relatesTo:
  - { to: concept.project, type: reads, note: "프로젝트 목록을 표시하고 전환을 수행한다" }
  - { to: concept.plan, type: reads, note: "활성 플랜 제목을 요약 영역에 표시한다" }
  - { to: concept.round, type: reads, note: "활성 라운드 번호와 모드를 표시한다" }
  - { to: concept.task, type: reads, note: "PlanTree에서 태스크를 탐색하고 선택할 수 있다" }
  - { to: screen.detail-drawer, type: leads-to, note: "항목 선택 시 디테일 드로어를 열어 상세를 표시한다" }
impacts:
  - concept.project
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

대시보드 좌측 고정 패널로, 사용자가 현재 작업 중인 프로젝트를 고르고, 그 프로젝트의 활성 플랜과 라운드를 즉시 파악하며, 플랜 계층을 트리로 탐색하는 데 쓰인다. 너비를 드래그로 조절하거나 접어서 공간을 확보할 수 있다. 접힌 상태에서는 프로젝트 이름 첫 글자 버튼으로 빠르게 프로젝트를 전환할 수 있다.

## UI 요소 / 입력 필드

- **브랜드 영역**: SDI 로고와 버전명, 사이드바 접기 토글 버튼.
- **프로젝트 스위처**: 현재 프로젝트 이름 버튼을 누르면 목록이 펼쳐진다. 비활성 프로젝트는 목록 하단에 표시되지만 선택 가능하다. 목록 하단에서 새 프로젝트 생성 모달을 열 수 있다.
- **활성 컨텍스트 요약**: 선택 프로젝트의 활성 플랜 제목과 활성 라운드(번호, 모드)를 한 줄씩 표시한다. 활성 플랜이 없으면 "No active plan"을 보여준다.
- **플랜 트리(PlanTree)**: 프로젝트의 플랜 계층을 트리로 펼친다. 각 플랜 아래 시나리오·요구사항·결정·라운드·태스크 항목을 탐색하고 선택할 수 있다. 플랜 헤더에서 플랜 생성 모달을 열 수 있다.
- **너비 조절 핸들**: 우측 가장자리 드래그로 폭을 조절하며, 조절 값과 접힘 여부는 로컬 저장소에 기억된다.
- **접힘 상태**: 로고 버튼과 프로젝트 첫 글자 아이콘만 세로로 나열되어 공간을 최소화한다.

## 표시 데이터 / 호출 API

프로젝트 목록은 앱 수준에서 주입된다. 선택 프로젝트가 바뀌거나 SSE 구조 이벤트가 오면, 해당 프로젝트의 플랜 목록과 라운드 목록을 데몬에서 새로 조회해 활성 컨텍스트 요약을 갱신한다. 활성 플랜은 status가 `active`인 첫 번째 플랜이고, 활성 라운드는 그 플랜에 속한 status `active` 라운드다.

## 상태 / 엣지케이스

- **접힘 상태**: 폭 48px로 고정되며 로고와 프로젝트 이니셜만 보인다.
- **프로젝트 없음**: "No projects yet" 또는 "Select a project" 안내를 표시한다.
- **플랜 없음**: PlanTree 영역에 빈 상태 안내와 새 플랜 만들기 버튼을 표시한다.
- **너비 한계**: 최소 200px, 최대 480px로 보정되며 로컬 저장소를 쓸 수 없으면 기본 288px로 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: PlanTree에서 시나리오·요구사항·결정·라운드·태스크의 표시 계층 구조(어떤 노드가 어떤 깊이에 나타나는지) 확인 필요.
