---
id: screen.patterns
kind: Screen
title: Patterns 뷰 (CollaborationPattern 거버넌스)
purpose: "소로 빌더 또는 코딩 에이전트가 선택 플랜의 CollaborationPattern 생애 주기, 결정 가역성 게이트, 활성 리소스 클레임을 4개 탭으로 검토한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/PatternsView.tsx
  - sdi-plugin/plugin/web/src/components/PatternTimeline.tsx
  - sdi-plugin/plugin/web/src/components/PatternTree.tsx
  - sdi-plugin/plugin/web/src/components/ReversibilityView.tsx
  - sdi-plugin/plugin/web/src/components/ClaimsLedger.tsx
consumesApi: []
relatesTo:
  - { to: concept.collaboration-pattern, type: reads, note: "패턴의 lifecycle(pending/active/converged/dissensus/aborted)과 kind, 계층 트리를 표시한다" }
  - { to: concept.plan, type: reads, note: "플랜 드롭다운으로 검토 대상 플랜을 전환한다" }
  - { to: concept.decision, type: reads, note: "Reversibility 탭에서 결정별 reversal_plan과 blast_radius_score를 표시한다" }
  - { to: concept.autonomy-policy, type: reads, note: "Reversibility 탭의 L5 잠금 해제 조건(blast_radius_score 임계값) 맥락에서 참조된다" }
  - { to: concept.disruption-review, type: reads, note: "Claims 탭에서 활성 시나리오 클레임 영역과 충돌 여부를 확인한다" }
  - { to: screen.detail-drawer, type: leads-to, note: "Reversibility 탭에서 결정을 선택하면 결정 디테일 드로어를 연다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

SDI v0.5 멀티에이전트 거버넌스 기능 전체를 한 뷰에서 모니터링하는 화면이다. 선택 플랜의 CollaborationPattern 이벤트 피드, 패턴 계층 트리, 결정 가역성 상태, 활성 리소스 클레임 현황을 각각 별도 탭으로 제공한다. 탑바의 활성 패턴 배지를 클릭하면 이 뷰로 이동한다.

## UI 요소 / 입력 필드

- **플랜 선택 드롭다운**: 프로젝트의 플랜 목록 중 검토할 플랜을 고른다. 기본값은 active 플랜이고 없으면 첫 번째 플랜이다.
- **4개 탭 내비게이션**: Timeline / Tree / Reversibility / Claims.
  - **Timeline 탭**: 선택 플랜의 CollaborationPattern 이벤트를 시간 순으로 나열한다. 각 패턴의 lifecycle 전환(pending→active→converged/dissensus/aborted)과 메타데이터를 보여준다.
  - **Tree 탭**: 패턴 간 parent_pattern_id 관계를 DAG 트리로 시각화한다. 각 노드에 kind·lifecycle·depth를 표시한다.
  - **Reversibility 탭**: 플랜에 속한 결정을 reversal_plan 존재 여부·blast_radius_score·L5 잠금 해제 상태별로 나열한다. 결정 클릭 시 결정 디테일 드로어를 열어 reversal_plan 전문을 확인할 수 있다.
  - **Claims 탭**: 플랜에 활성화된 시나리오 리소스 클레임(claimed_resources_json의 경로 글로브) 목록과 claim_status를 표시한다.

## 표시 데이터 / 호출 API

화면 진입 시 프로젝트의 플랜 목록을 조회해 기본 선택 플랜을 결정한다. 이후 각 탭은 선택된 플랜 ID를 기준으로 관련 데이터를 각각의 데몬 엔드포인트에서 조회한다. 플랜 전환이나 refreshKey 변경 시 각 탭 컴포넌트가 데이터를 다시 가져온다.

## 상태 / 엣지케이스

- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **플랜 없음**: 플랜 드롭다운에 "(no plans)" 옵션을 표시하고 탭 내용을 비운다.
- **선택 플랜 없음**: "Select a plan to see its patterns." 안내를 표시한다.
- **탭별 빈 상태**: 각 탭 컴포넌트가 자체적으로 빈 상태를 처리한다.
- **direct 안티패턴**: kind=direct인 패턴은 탑바 배지에 빨간 점으로 표시되며, Tree 탭에서도 별도로 강조된다.

## 미확정 (OPEN)
- [ ] OPEN: PatternTree의 depth 시각화 방식과 DAG 사이클 방지 렌더링 처리 방식 확인 필요.
