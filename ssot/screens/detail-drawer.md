---
id: screen.detail-drawer
kind: Screen
title: 디테일 드로어 (항목 상세 오버레이)
purpose: "소로 빌더가 플랜·시나리오·요구사항·결정·라운드·태스크 중 하나를 선택하면 화면 우측 오버레이 패널에서 상세를 보고 상태를 변경하거나 증거를 기록한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/App.tsx
  - sdi-plugin/plugin/web/src/components/PlanDetail.tsx
  - sdi-plugin/plugin/web/src/components/TaskDetail.tsx
  - sdi-plugin/plugin/web/src/components/ScenarioDetail.tsx
  - sdi-plugin/plugin/web/src/components/RequirementDetail.tsx
  - sdi-plugin/plugin/web/src/components/DecisionDetail.tsx
  - sdi-plugin/plugin/web/src/components/RoundDetail.tsx
consumesApi: []
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜 선택 시 시나리오·요구사항·결정·라운드 목록과 AutonomyPolicy를 함께 표시한다" }
  - { to: concept.scenario, type: reads, note: "시나리오 선택 시 GWT 상세를 표시하고 confirm 동작을 수행한다" }
  - { to: concept.task, type: reads, note: "태스크 선택 시 설명·상태·연결 시나리오·증거를 표시하고 상태 전환 및 증거 제출을 수행한다" }
  - { to: concept.task-evidence, type: reads, note: "태스크 완료 시 각 시나리오별 결과(passing/failing/impacted/retired)와 증거 참조를 입력한다" }
  - { to: concept.decision, type: reads, note: "결정 선택 시 D20 4단계 합의 흐름, reversal_plan, blast_radius_score를 표시한다" }
  - { to: concept.round, type: reads, note: "라운드 선택 시 모드·상태와 라운드 관련 동작을 표시한다" }
  - { to: concept.autonomy-policy, type: reads, note: "플랜 상세 안의 AutonomyPanel에서 플랜의 자율성 정책을 표시한다" }
  - { to: concept.agent-note, type: reads, note: "플랜 상세 안의 AgentNotesPanel에서 에이전트 메모를 표시한다" }
impacts:
  - concept.plan
  - concept.task
  - concept.scenario
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

사이드바 PlanTree 또는 각 뷰(Board, Backlog 등)에서 항목을 선택하면 화면 우측에서 슬라이드로 열리는 오버레이 패널이다. 항목 종류(플랜·시나리오·요구사항·결정·라운드·태스크)에 따라 다른 상세 컴포넌트가 렌더된다. 배경을 클릭하거나 닫기 버튼, Esc 키로 닫는다. 좌측 가장자리 드래그로 너비를 조절하며 로컬 저장소에 기억된다.

## UI 요소 / 입력 필드

드로어 내용은 선택된 항목 종류에 따라 달라진다.

**플랜 상세**: 플랜 코드·제목·상태 배지, "Approve plan"·"Complete plan"·"+ Round" 동작 버튼, 마크다운 본문, 시나리오 목록(GWT + confirm 버튼), 요구사항 목록, 결정 타임라인, AutonomyPanel, AgentNotesPanel, 라운드 목록.

**태스크 상세**: 태스크 코드·상태 배지, 설명, 상태 전환 버튼(todo/in_progress/blocked/done/cancelled), 연결 시나리오의 GWT, 증거 입력 폼(각 시나리오별 결과·증거 참조·비고 + 전체 요약), 기 제출된 증거 표시.

**시나리오·요구사항·결정·라운드**: 각각 해당 엔티티의 상세 필드와 관련 동작 버튼을 표시한다.

## 표시 데이터 / 호출 API

항목이 선택되면 해당 엔티티 ID로 데몬에서 상세 데이터를 조회한다. 플랜 상세는 시나리오·요구사항·결정·라운드를 병렬로 추가 조회한다. 태스크 상세는 태스크 데이터와 각 parent_scenario_id로 시나리오 상세를 조회한다. 상태 변경·승인·완료·증거 제출 등의 쓰기 동작은 해당 엔티티의 데몬 엔드포인트를 호출하며, 성공 시 토스트 알림을 표시한다. 실패 시 에러 코드와 메시지를 토스트로 표시한다.

## 상태 / 엣지케이스

- **닫힘 상태**: 선택 항목이 없으면 드로어가 화면 밖에 위치하며 클릭이 통하지 않는다.
- **로딩 중**: 데이터 조회 전 "Loading…" 텍스트를 표시한다.
- **에러**: 데이터 조회 실패 시 에러 메시지를 표시한다.
- **태스크 done 전환 시 증거 필요**: 데몬이 EVIDENCE_REQUIRED 에러를 반환하면 증거 입력 폼을 자동으로 열고 "Evidence required" 토스트를 표시한다.
- **플랜 승인 조건**: confirmed 시나리오가 하나 이상 있어야 "Approve plan" 버튼이 활성화된다.
- **배경 클릭으로 닫기**: 드로어 외부(배경 오버레이) 클릭 시 선택 해제로 닫힌다.
- **너비 한계**: 최소 360px, 최대 90vw 범위로 보정되며 로컬 저장소를 쓸 수 없으면 기본 520px로 동작한다.

## 미확정 (OPEN)
- [ ] OPEN: RoundDetail 컴포넌트에서 제공하는 동작(라운드 activate·complete 등)의 구체 UI 확인 필요.
