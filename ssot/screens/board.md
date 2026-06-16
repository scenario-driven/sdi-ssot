---
id: screen.board
kind: Screen
title: Board 뷰 (라운드 칸반 보드)
purpose: "소로 빌더가 활성 플랜의 활성 라운드에 속한 태스크를 칸반 보드에서 보고, 드래그앤드롭으로 상태를 변경하며 진행 상황을 추적한다."
servesPersona:
  - persona.solo-builder
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/BoardView.tsx
consumesApi: []
relatesTo:
  - { to: concept.plan, type: reads, note: "활성 플랜을 찾아 보드 스코프를 결정한다" }
  - { to: concept.round, type: reads, note: "활성 라운드(또는 첫 번째 라운드)에 속한 태스크를 보드에 표시한다" }
  - { to: concept.task, type: reads, note: "태스크를 상태별 컬럼에 배치하고 DnD로 상태를 변경한다" }
impacts:
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

활성 플랜과 활성 라운드(없으면 가장 첫 번째 라운드)에 속한 태스크를 5개 컬럼 칸반으로 시각화하는 뷰다. 태스크 카드를 다른 컬럼으로 끌어다 놓으면 즉시 데몬에 상태 변경을 요청하고 낙관적으로 화면을 갱신한다. SSE 이벤트를 통해 다른 에이전트·세션이 변경한 태스크도 실시간으로 반영된다.

## UI 요소 / 입력 필드

입력 필드는 없다. 모든 조작은 카드 드래그앤드롭으로 이루어진다.

- **헤더**: 현재 활성 플랜 코드, 라운드 코드, 라운드 모드를 한 줄로 보여준다.
- **5개 컬럼(칸반)**: todo / in_progress / blocked / done / cancelled. 각 컬럼에는 컬럼 이름과 태스크 수가 표시된다. 태스크를 다른 컬럼으로 끌어다 놓으면 상태가 바뀐다.
- **태스크 카드**: short_code와 설명(최대 3줄)을 표시한다. 카드를 클릭하면 디테일 드로어가 열린다.
- **드래그 오버레이**: 드래그 중인 카드의 고스트 이미지가 커서를 따라다닌다.

## 표시 데이터 / 호출 API

화면 진입 시 플랜 목록 → 활성 플랜 선택 → 라운드 목록 → 활성(또는 첫 번째) 라운드 선택 → 해당 라운드의 태스크 목록 순서로 데몬을 조회한다. 이후 SSE로 수신되는 태스크 패치(부분 업데이트)는 재조회 없이 인메모리에서 병합해 즉시 반영한다. 태스크를 다른 컬럼으로 옮기면 데몬의 태스크 상태 변경 기능을 호출한다. 실패하면 이전 상태로 롤백하고 에러 토스트를 표시한다.

## 상태 / 엣지케이스

- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **활성 플랜 없음**: "No active plan in this project. Activate a plan to populate the board." 안내를 표시한다.
- **라운드 없음**: "Plan has no rounds yet." 안내를 표시한다.
- **컬럼 빈 상태**: 태스크가 없는 컬럼도 드롭 영역으로 유지된다.
- **DnD 실패**: 낙관적으로 변경된 상태를 원래대로 되돌리고 에러 토스트를 표시한다.
- **드래그 중 컬럼 진입**: 대상 컬럼에 primary 색상 ring이 표시된다.

## 미확정 (OPEN)
- [ ] OPEN: 라운드가 여러 개 있을 때 active 라운드가 없는 경우 첫 번째 라운드를 선택하는 규칙이 의도된 것인지 확인 필요.
