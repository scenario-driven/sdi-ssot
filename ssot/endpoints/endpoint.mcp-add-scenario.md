---
id: endpoint.mcp-add-scenario
kind: Endpoint
title: MCP add_scenario — GWT 시나리오 추가
definition: LLM 에이전트가 활성 플랜에 Given/When/Then 시나리오를 추가하는 MCP 쓰기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/write.rs
relatesTo:
  - to: concept.scenario
    type: mutates
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

`add_scenario` 는 SDI MCP 서버가 노출하는 쓰기 도구 중 하나다. Claude Code 내부의 LLM 에이전트가 계획 수립 또는 시나리오 설계 단계에서 특정 플랜에 새 GWT 시나리오를 등록할 때 호출한다. MCP 레이어는 입력을 받아 SDI 데몬의 REST 엔드포인트(`POST /plans/:id/scenarios`)로 위임하며, 구조 검증과 중복 여부 판단은 데몬이 최종 수행한다.

이 도구는 단순한 데이터 저장 이상의 의미를 갖는다. SDI 에서 시나리오는 구현의 1등 시민이며, 등록된 시나리오는 이후 태스크 분해와 증거 수집의 기준점이 된다. 따라서 에이전트가 이 도구를 호출할 때는 구현 가능한 수준으로 구체화된 행동 명세를 전달해야 한다.

## 요청 / 응답

**입력이 의미하는 것**

에이전트는 다섯 가지 핵심 정보를 전달한다. 첫째는 어느 플랜에 추가할지 특정하는 플랜 ID다. 둘째는 시나리오를 한 문장으로 요약한 제목으로, 팀이 이 시나리오를 식별할 때 사용하는 이름이 된다. 셋째부터 다섯째는 GWT 세 단계다. Given은 이 시나리오가 성립하기 위한 사전 조건(시스템 상태, 데이터, 환경 설정 등)을 기술하고, When은 사용자나 시스템이 취하는 단일 행동 또는 이벤트를 기술하며, Then은 그 행동의 결과로 관찰되어야 하는 기대 동작을 기술한다. 선택적으로 태그 목록을 함께 전달하면 이후 검색과 필터링에 활용된다.

**출력이 의미하는 것**

호출이 성공하면 데몬이 생성한 시나리오 ID와 등록 확인 메시지를 반환한다. 이 ID는 이후 `update_task_evidence` 호출 시 시나리오 참조에 사용되므로 에이전트가 세션 컨텍스트에 보관해야 한다.

## 권한 / 제약

이 도구를 호출하려면 대상 플랜이 반드시 `active` 상태여야 한다. 아직 승인되지 않은 `draft` 플랜이나 완료된 플랜에는 시나리오를 추가할 수 없으며, 데몬이 이를 거부한다.

Given, When, Then 세 필드는 모두 필수다. D5 엄격 검증 규칙에 따라 셋 중 하나라도 비어 있으면 데몬이 즉시 오류를 반환한다. 빈 문자열이나 공백만 있는 경우도 비어 있는 것으로 간주된다.

동일한 제목의 시나리오가 같은 플랜에 이미 존재하는 경우 데몬이 중복으로 판단하여 거부할 수 있다. 에이전트는 등록 전에 `search_scenarios` 도구로 유사 시나리오가 없는지 먼저 확인하는 것이 권장된다.

MCP 서버는 SDI 데몬이 `~/.cache/sdi/sdid.port` 에 기록한 포트로 HTTP 요청을 전달한다. 데몬이 실행 중이지 않으면 이 도구를 포함한 모든 MCP 도구가 동작하지 않는다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/write.rs` 구현 분석에서 추론
- 위임 엔드포인트: `POST /plans/:id/scenarios` (데몬 REST API)
- 관련 설계 결정: D5 (GWT 필수 3단계 검증), D12 (스냅샷 의미론)

## 미확정 (OPEN)

- 태그 목록의 최대 개수와 각 태그 문자열의 길이 제한이 명시되어 있지 않음
- 중복 시나리오 감지 기준이 제목 완전 일치인지 유사도 기반인지 미확인
- MCP 레이어 자체의 입력 사전 검증 범위(데몬 위임 전)가 코드 확인 전까지 불명확
- 시나리오 생성 시 초기 상태가 `draft`인지 `confirmed`인지 미확인
