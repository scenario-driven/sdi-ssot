---
id: screen.summary
kind: Screen
title: Summary 뷰 (프로젝트 스냅샷)
purpose: "소로 빌더 또는 코딩 에이전트가 현재 프로젝트의 플랜·라운드·시나리오·태스크 상태를 한 페이지 요약으로 파악한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/SummaryView.tsx
consumesApi: []
relatesTo:
  - { to: concept.plan, type: reads, note: "전체 플랜 수, 활성 플랜 short_code와 제목을 카드로 표시한다" }
  - { to: concept.round, type: reads, note: "활성 라운드 short_code와 모드를 카드로 표시한다" }
  - { to: concept.scenario, type: reads, note: "전체 시나리오 수와 confirmed/draft 분포를 표시한다" }
  - { to: concept.task, type: reads, note: "활성 라운드의 태스크를 상태별로 집계해 표시한다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

프로젝트에 진입하거나 현재 상태를 빠르게 점검할 때 가장 먼저 보는 뷰다. 활성 플랜이 있는 경우 그 플랜의 시나리오·라운드·태스크 현황을 숫자 카드와 목록으로 보여준다. "지금 무엇이 진행 중이고, 얼마나 됐는가"를 한 화면에서 답한다.

## UI 요소 / 입력 필드

요약 화면은 읽기 전용이다. 별도의 입력 필드는 없다.

- **상단 4개 지표 카드**: 전체 플랜 수 / 활성 플랜 식별자 및 제목 / 활성 라운드 식별자 및 모드 / 시나리오 총수와 confirmed·draft 분포.
- **라운드 태스크 현황 바**: 활성 라운드가 있을 때 todo·in_progress·blocked·done·cancelled 5개 상태별 태스크 수를 작은 카드 5개로 표시한다. 활성 라운드가 없으면 "No active round — activate one to start tracking tasks" 안내를 보여준다.
- **플랜 목록**: 프로젝트의 모든 플랜을 줄로 나열하며 short_code·제목·상태 배지(draft/active/completed)를 표시한다.

## 표시 데이터 / 호출 API

화면이 열리거나 SSE 구조 이벤트가 발생하면 다음 순서로 데이터를 조회한다. 먼저 프로젝트의 플랜 목록을 가져오고, 활성 플랜이 있으면 그 플랜의 라운드 목록과 시나리오 목록을 동시에 조회한다. 활성 라운드가 있으면 해당 라운드의 태스크 목록을 추가로 조회한다. 조회가 모두 끝나면 집계 결과를 화면에 반영한다.

## 상태 / 엣지케이스

- **로딩 중**: "Loading…" 텍스트를 표시한다.
- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **활성 플랜 없음**: 플랜 카드에 "—"와 "No active plan"을 표시한다.
- **활성 라운드 없음**: 라운드 카드에 "—"를 표시하고, 태스크 현황 영역에 "No active round" 안내를 표시한다.
- **플랜 없음**: 플랜 목록에 "No plans yet" 텍스트를 표시한다.

## 미확정 (OPEN)
- [ ] OPEN: 여러 플랜이 동시에 active인 경우(정책상 가능한지) 활성 플랜 카드가 어떻게 처리되는지 확인 필요.
