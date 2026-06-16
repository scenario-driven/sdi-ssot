---
id: screen.next
kind: Screen
title: Next 뷰 (다음 기계적 단계)
purpose: "소로 빌더 또는 코딩 에이전트가 데몬이 계산한 다음 실행 명령, 재검토해야 할 provisional 결정 목록, 그리고 현재 in_progress 태스크의 위임 브리프를 한 화면에서 확인한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/NextView.tsx
consumesApi: []
relatesTo:
  - { to: concept.task, type: reads, note: "in_progress 태스크의 위임 브리프(설명·연결 시나리오·증거 형식·금지사항)를 표시한다" }
  - { to: concept.decision, type: reads, note: "provisional 상태로 표시된 결정을 재검토 목록으로 나열한다" }
  - { to: concept.scenario, type: reads, note: "태스크 브리프 안에서 연결된 시나리오의 GWT를 표시한다" }
  - { to: concept.task-evidence, type: reads, note: "브리프의 evidence_format과 complete_with 명령을 안내한다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

SDI 워크플로에서 "지금 당장 무엇을 실행해야 하는가"를 단 하나의 명령으로 제시하는 뷰다. 데몬이 현재 프로젝트의 상태(플랜·라운드·태스크 진행 상황)를 분석해 다음 CLI 명령과 그 이유를 계산해 돌려주며, 추가로 재검토 대상 provisional 결정 목록과 현재 진행 중 태스크의 위임 브리프를 보여준다. 코딩 에이전트 또는 서브에이전트가 "어디서 시작하면 되는가"를 즉시 파악하는 데 최적화된 화면이다.

## UI 요소 / 입력 필드

읽기 전용 화면이며 입력 필드는 없다.

- **다음 단계 카드**: 데몬이 계산한 CLI 명령(모노스페이스 코드 블록)과 그 이유를 한 카드에 보여준다.
- **Provisional 결정 재검토 목록**: provisional 상태 결정이 하나 이상 있으면 경고 스타일의 항목 목록으로 표시한다. 각 항목에는 short_code, 제목, `supersede_when` 조건이 나온다.
- **태스크 위임 브리프 카드**: 다음 단계 명령이 in_progress 태스크를 가리키는 경우(`sdi task brief <id>` 형태), 그 태스크의 상세 브리프를 별도 카드로 표시한다. 브리프에는 태스크 설명, 연결 시나리오의 GWT, 검증 기준선, 증거 형식, 완료 명령, 금지사항 목록이 포함된다.

## 표시 데이터 / 호출 API

화면이 열리거나 refreshKey가 변경되면 데몬의 "next step" 조회 기능을 호출해 다음 단계 정보를 가져온다. 응답 안에 in_progress 태스크 브리프 명령이 포함된 경우, 추가로 해당 태스크의 브리프를 조회한다.

## 상태 / 엣지케이스

- **로딩 중**: "Loading…" 텍스트를 표시한다.
- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **Provisional 결정 없음**: 재검토 섹션 자체를 숨긴다.
- **태스크 브리프 없음**: 브리프 카드를 표시하지 않는다.
- **브리프 조회 실패**: 에러를 조용히 무시하고 브리프 카드를 표시하지 않는다(null 처리).

## 미확정 (OPEN)
- [ ] OPEN: 데몬이 next step을 계산하지 못할 경우(플랜/라운드 없음 등)의 응답 구조와 화면 처리 확인 필요.
