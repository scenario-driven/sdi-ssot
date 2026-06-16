---
id: endpoint.mcp-get-plan-context
kind: Endpoint
title: MCP get_plan_context — 플랜 전체 컨텍스트 조회
definition: LLM 에이전트가 플랜 메타데이터, 시나리오, 진행 중 태스크, 최근 결정을 단일 호출로 조회하는 MCP 읽기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/read.rs
relatesTo:
  - to: concept.plan
    type: reads
  - to: concept.scenario
    type: reads
  - to: concept.task
    type: reads
  - to: concept.decision
    type: reads
  - to: domain.planning
    type: belongs-to
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`get_plan_context` 는 SDI MCP 서버의 읽기 도구 중에서 가장 폭넓은 정보를 반환하는 도구다. LLM 에이전트가 새 작업 세션을 시작하거나 현재 상황을 전반적으로 파악해야 할 때 사용한다. 내부적으로 플랜 메타데이터(`GET /plans/:id`), 시나리오 목록, 진행 중 태스크 목록, 최근 결정 목록을 조합하여 단일 응답으로 구성한다.

이 도구의 핵심 가치는 "한 번의 호출로 전체 그림을 본다"는 데 있다. 에이전트가 개별 도구를 여러 번 호출하지 않고도 "지금 이 플랜에서 무슨 일이 일어나고 있는가"를 파악할 수 있다. 그 대신 응답 크기가 크고, MCP 레이어의 응답 상한을 초과할 경우 데이터가 잘릴 수 있다는 트레이드오프가 있다.

## 요청 / 응답

**입력이 의미하는 것**

플랜 ID를 전달하면 해당 플랜의 전체 컨텍스트를 조회한다. 플랜 ID를 생략하면 데몬이 현재 활성 플랜을 자동으로 탐지하여 사용한다.

**출력이 의미하는 것**

응답은 네 층위의 정보를 담는다.

첫째, 플랜 메타데이터: 플랜의 제목, 현재 상태(draft/active/completed), 승인 여부, 생성일시, 담당자 정보가 포함된다. 에이전트는 이를 통해 이 플랜이 현재 작업 가능한 상태인지를 즉시 판단할 수 있다.

둘째, 시나리오 목록: 플랜에 등록된 모든 시나리오를 상태별(confirmed/draft/retired)로 구분하여 반환한다. 각 시나리오의 GWT 요약이 포함되어 에이전트가 구현 범위를 빠르게 파악할 수 있다.

셋째, 진행 중인 태스크 목록: 현재 `in_progress` 또는 `todo` 상태인 태스크들과 각 태스크의 담당자, 연결된 시나리오 수가 포함된다. 에이전트는 이를 통해 자신이 이어받을 작업이나 협업 중인 작업을 파악한다.

넷째, 최근 결정 요약: 가장 최근에 기록된 결정 몇 건의 제목과 상태를 요약하여 포함한다. 전체 결정 이력이 필요하면 `get_recent_decisions` 도구를 별도로 호출한다.

## 권한 / 제약

이 도구는 읽기 전용이며 상태를 변경하지 않는다.

응답 크기가 MCP 레이어의 응답 상한(invariant.mcp-response-cap 추정)을 초과하는 경우 데이터가 잘릴 수 있다. 시나리오가 많고 태스크가 많은 플랜에서는 이 도구 대신 각 세부 도구(`search_scenarios`, `get_recent_decisions` 등)를 목적에 맞게 개별 호출하는 것이 더 안정적이다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/read.rs` 구현 분석에서 추론
- 위임 엔드포인트: `GET /plans/:id` 및 하위 엔드포인트 조합 (데몬 REST API)
- 관련 설계 결정: D6 (라운드 기반 컨텍스트), MCP 레이어 응답 상한 invariant

## 미확정 (OPEN)

- 응답 상한 초과 시 잘림 기준이 어느 층위(시나리오/태스크/결정 중 어느 것을 먼저 자르는지)가 불명확
- 활성 플랜 자동 탐지 로직이 단일 active 플랜을 전제하는지, 복수 시 선택 기준이 있는지 미확인
- "진행 중 태스크 목록"에 포함되는 상태 범위(`todo`만인지, `blocked` 포함인지)가 불명확
- 시나리오 GWT 요약의 길이 및 요약 방식(전문 vs. 잘림)이 불명확
