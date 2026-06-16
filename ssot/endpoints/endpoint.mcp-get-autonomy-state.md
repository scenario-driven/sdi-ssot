---
id: endpoint.mcp-get-autonomy-state
kind: Endpoint
title: MCP get_autonomy_state — 자율성 정책 스냅샷 조회
definition: LLM 에이전트가 현재 플랜에 적용되는 자율성 레벨과 허용 행동 범위를 조회하는 MCP 읽기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/read.rs
relatesTo:
  - to: concept.autonomy-policy
    type: reads
  - to: concept.plan
    type: reads
  - to: domain.autonomy
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

`get_autonomy_state` 는 SDI MCP 서버의 읽기 도구로, LLM 에이전트가 작업을 시작하기 전 자신이 얼마나 자율적으로 행동할 수 있는지를 파악할 때 사용한다. 데몬의 `GET /autonomy_policies/resolve` 엔드포인트로 위임되며, 플랜 ID(또는 현재 활성 플랜 자동 탐지)를 기준으로 현재 적용되는 자율성 정책을 반환한다.

SDI 의 자율성 레벨은 L3(제한적 자율), L4(협의 후 자율), L5(완전 자율)로 구분된다. D14/D17 설계 결정에 따라 각 레벨은 에이전트가 사람의 승인 없이 실행 가능한 행동 집합을 정의한다. 에이전트는 이 도구를 통해 자신이 처한 정책 환경을 먼저 파악하고, 허용 범위 안에서만 행동해야 한다.

## 요청 / 응답

**입력이 의미하는 것**

플랜 ID를 전달하면 해당 플랜에 설정된 자율성 정책을 조회한다. 플랜 ID를 생략하면 데몬이 현재 활성 플랜을 자동으로 탐지하여 적용 정책을 반환한다.

**출력이 의미하는 것**

응답은 현재 플랜에 실제로 적용되는 자율성 레벨(L3/L4/L5), 그 레벨의 의미를 설명하는 정책 설명문, circuit-breaker 발동 여부와 그 이유, 그리고 현재 레벨에서 에이전트가 할 수 있는 행동 목록과 할 수 없는 행동 목록을 포함한다. 에이전트는 이 응답을 바탕으로 사람에게 승인을 요청해야 하는 행동과 독립적으로 진행 가능한 행동을 판단한다.

## 권한 / 제약

이 도구는 읽기 전용이며 상태를 변경하지 않는다.

정책이 명시적으로 설정되지 않은 플랜에 대해서는 데몬이 기본 레벨(L3 추정)을 적용하여 응답한다.

D17 강제 L4 플래그가 설정된 플랜에서는 레벨이 L4 아래로 내려갈 수 없다. 즉 L3 정책이 명시되어 있더라도 D17 플래그가 있으면 실제 적용 레벨은 L4로 올라간다. 이 도구의 응답은 이미 이 플래그를 반영한 최종 결정 레벨을 반환한다.

circuit-breaker 상태가 활성화되어 있으면 레벨과 무관하게 에이전트의 자율 행동 범위가 축소될 수 있으며, 응답에 그 이유가 기재된다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/read.rs` 구현 분석에서 추론
- 위임 엔드포인트: `GET /autonomy_policies/resolve` (데몬 REST API)
- 관련 설계 결정: D14 (자율성 레벨 정의), D17 (강제 L4 플래그)

## 미확정 (OPEN)

- 기본 레벨이 L3인지, 다른 값인지 코드로 확인 필요
- circuit-breaker 발동 조건의 전체 목록이 불명확
- L3/L4/L5 각 레벨의 정확한 허용 행동 목록이 SSOT에 아직 없음 — `concept.autonomy-policy` 노드로 분리 필요
- 플랜 ID 없이 활성 플랜을 탐지하는 로직(단일 active 플랜 전제인지, 가장 최근 플랜인지)이 불명확
