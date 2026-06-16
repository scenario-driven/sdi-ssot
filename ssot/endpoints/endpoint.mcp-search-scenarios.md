---
id: endpoint.mcp-search-scenarios
kind: Endpoint
title: MCP search_scenarios — 플랜 내 시나리오 전문 검색
definition: LLM 에이전트가 플랜 내 등록된 시나리오를 키워드로 검색하여 중복 여부 확인 및 관련 케이스 탐색에 사용하는 MCP 읽기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/read.rs
relatesTo:
  - to: concept.scenario
    type: reads
  - to: concept.plan
    type: reads
  - to: domain.scenario-management
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

`search_scenarios` 는 SDI MCP 서버의 읽기 도구로, LLM 에이전트가 특정 플랜 내에 등록된 시나리오들을 전문 검색으로 탐색할 때 사용한다. 데몬의 `GET /plans/:id/scenarios?q=...` 엔드포인트로 위임된다.

이 도구는 주로 두 가지 목적으로 사용된다. 첫째, 새 시나리오를 추가(`add_scenario`)하기 전 유사 시나리오가 이미 존재하는지 확인하는 중복 검사다. 둘째, 특정 기능이나 동작과 관련된 기존 시나리오들을 찾아 구현 범위를 파악하거나 태스크 분해에 활용하는 탐색이다. 시나리오 제목과 GWT 내용 모두를 대상으로 검색이 이루어진다.

## 요청 / 응답

**입력이 의미하는 것**

플랜 ID는 필수다. 이 도구는 플랜 범위 내에서만 검색하므로, 어느 플랜을 대상으로 할지 명시해야 한다. 검색어는 시나리오 제목, Given 조건, When 행동, Then 기대 결과 등 GWT 어느 부분에서도 매칭될 수 있다.

**출력이 의미하는 것**

응답은 검색어와 일치하는 시나리오 목록이다. 각 항목은 시나리오의 제목, GWT 세 단계의 요약, 현재 상태(confirmed/draft/retired), 그리고 이 시나리오가 특정 태스크에 의해 클레임된 상태인지 여부를 포함한다.

에이전트는 이 정보를 통해 중복 시나리오를 피하고, 이미 클레임된 시나리오는 다른 에이전트가 작업 중임을 파악하며, `retired` 상태 시나리오는 더 이상 유효하지 않다는 것을 인지한다. `confirmed` 상태 시나리오만이 구현 기준이 된다.

## 권한 / 제약

이 도구는 읽기 전용이며 시나리오의 상태를 변경하지 않는다.

플랜 ID는 필수 파라미터다. 전체 프로젝트에 걸친 시나리오 전역 검색은 이 도구로 불가능하다. 특정 플랜 범위로 제한된 설계는 의도적이며, 에이전트가 현재 작업 중인 플랜 맥락 안에서만 시나리오를 탐색하도록 유도한다.

존재하지 않는 플랜 ID를 지정하면 데몬이 오류를 반환한다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/read.rs` 구현 분석에서 추론
- 위임 엔드포인트: `GET /plans/:id/scenarios?q=...` (데몬 REST API)
- 관련 설계 결정: D5 (GWT 필수 검증), 시나리오 상태 관리(confirmed/draft/retired)

## 미확정 (OPEN)

- 검색 결과의 최대 반환 수와 관련성 정렬 기준이 불명확
- GWT 각 단계가 검색 인덱스에 동일한 가중치로 포함되는지, 제목에 더 높은 가중치가 주어지는지 미확인
- `retired` 상태 시나리오가 기본적으로 검색 결과에 포함되는지, 필터 파라미터가 있는지 불명확
- "클레임 여부"가 태스크와의 연결을 나타내는 것인지, 별도의 클레임 메커니즘이 있는지 미확인
