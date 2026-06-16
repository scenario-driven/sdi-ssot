---
id: endpoint.mcp-get-recent-decisions
kind: Endpoint
title: MCP get_recent_decisions — 최근 결정 목록 조회
definition: LLM 에이전트가 플랜의 최신 의사결정 기록(ADR 로그)을 최신순으로 조회하는 MCP 읽기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/read.rs
relatesTo:
  - to: concept.decision
    type: reads
  - to: concept.plan
    type: reads
  - to: domain.decision-negotiation
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

`get_recent_decisions` 는 SDI MCP 서버의 읽기 도구로, LLM 에이전트가 특정 플랜에서 내려진 최근 의사결정들을 시계열 순서로 조회할 때 사용한다. 데몬의 `GET /plans/:id/decisions` 엔드포인트로 위임되며, 가장 최근에 기록된 결정부터 역순으로 정렬된 결과를 반환한다.

이 도구는 에이전트가 작업에 착수하기 전 "이 플랜에서 어떤 결정들이 이미 내려졌는가"를 확인하는 데 주로 사용된다. 중복 결정을 피하고, 이미 확정된 방향을 재논의하는 낭비를 막으며, superseded 상태의 결정을 통해 방향이 어떻게 진화해왔는지 파악할 수 있다.

## 요청 / 응답

**입력이 의미하는 것**

플랜 ID는 필수다. 어느 플랜의 결정 이력을 볼지를 명시한다. 선택적으로 최대 반환 수를 지정할 수 있으며, 생략하면 데몬이 정의한 기본값이 적용된다.

**출력이 의미하는 것**

응답은 결정 항목의 배열이다. 각 항목은 다음 정보를 담는다. 결정 제목은 한 줄로 결정의 핵심을 나타낸다. 결정 내용은 배경, 선택지, 근거를 포함한 전문이다. 상태는 이 결정이 현재 유효한지(`confirmed`), 잠정적인지(`provisional`), 이미 다른 결정으로 대체되었는지(`superseded`)를 나타낸다. 생성일시는 이 결정이 언제 내려졌는지를 보여준다. 대체 체인 정보는 이 결정이 어떤 이전 결정을 대체했는지(`supersedes_id`), 또는 이 결정이 어떤 이후 결정에 의해 대체되었는지를 나타낸다.

에이전트는 `superseded` 상태의 결정을 현재 유효한 결정으로 오해해서는 안 된다. 최신 `confirmed` 결정만이 현재 플랜의 확정된 방향이다.

## 권한 / 제약

이 도구는 읽기 전용이며 상태를 변경하지 않는다.

MCP 레이어의 응답 상한(invariant.mcp-response-cap 추정)이 적용되므로, 결정이 많이 쌓인 플랜에서는 오래된 결정이 응답에서 잘릴 수 있다. 전체 이력이 필요한 경우 최대 반환 수를 명시적으로 줄여서 여러 번 호출하거나, 데몬 REST API에 직접 접근하는 방법을 고려해야 한다.

플랜 ID가 존재하지 않으면 데몬이 오류를 반환한다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/read.rs` 구현 분석에서 추론
- 위임 엔드포인트: `GET /plans/:id/decisions` (데몬 REST API)
- 관련 설계 결정: append-only 결정 이력 원칙, supersedes 체이닝 구조

## 미확정 (OPEN)

- 기본 반환 수(max 지정 생략 시 몇 건인지)가 불명확
- 페이지네이션 지원 여부가 불명확 — 현재는 offset/cursor 파라미터가 없는 것으로 추정
- `provisional` 상태 결정이 별도 표시 없이 `confirmed` 결정과 동일 목록에 혼합되는지, 구분 표시되는지 미확인
- supersedes 체인 전체를 한 번에 조회할 수 있는지, 아니면 개별 결정 ID로 체인을 직접 추적해야 하는지 미확인
