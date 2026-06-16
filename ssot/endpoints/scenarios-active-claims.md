---
id: endpoint.scenarios-active-claims
kind: Endpoint
title: "GET /scenarios/active-claims"
definition: >
  D29: 현재 claim_status가 active인 모든 시나리오를 반환하는 엔드포인트.
  선택적으로 plan_id로 범위를 좁힐 수 있다. PreToolUse 훅이 이 엔드포인트를 조회하여
  Edit/Write 허용 전 파일 경로 글로브 중복을 사전에 탐지한다.
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/scenario.rs"
relatesTo:
  - to: concept.scenario-claim
    type: reads
  - to: concept.scenario
    type: reads
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

데몬이 현재 추적하고 있는 모든 활성 파일 경로 클레임 목록을 반환한다. `claim_status`가 `active`인 시나리오만 응답에 포함되며, released 또는 초기 상태의 시나리오는 제외된다.

`plan_id` 쿼리 파라미터를 지정하면 해당 플랜에 속하는 활성 클레임만 반환한다. 파라미터 없이 호출하면 데몬 전체(모든 플랜)에 걸친 활성 클레임 대장(ledger)을 반환한다.

주된 사용처는 Claude Code의 PreToolUse 훅이다. 훅은 Edit 또는 Write 도구 호출이 발생하기 직전에 이 엔드포인트를 조회하고, 작업 대상 파일이 이미 다른 시나리오나 세션에 의해 클레임된 경로 글로브와 겹치는지 확인한다. 겹칠 경우 도구 호출을 차단하여 교차 세션 파일 충돌을 방지한다.

## 요청 / 응답

- 메서드: GET
- 쿼리 파라미터:
  - `plan_id` (선택): 특정 플랜으로 결과를 좁힐 때 지정
- 요청 본문: 없음
- 응답: 활성 클레임 시나리오 목록. 각 항목에는 시나리오 식별자, 선언된 파일 경로 글로브 패턴, 클레임 등록 시각, 클레임 등록 세션 식별자(존재하는 경우) 등이 포함될 것으로 추정됨
- 활성 클레임이 없으면 빈 배열 반환

## 권한 / 제약

- 별도 인증 계층 없이 로컬 데몬에 직접 접근한다.
- 읽기 전용이므로 어떠한 상태도 변경하지 않는다.
- 이 엔드포인트의 응답은 PreToolUse 훅이 충돌 감지 결정을 내리는 근거 데이터다. 응답 지연이 훅 실행 속도에 직접 영향을 미치므로 응답 지연이 최소화되어야 한다.
- 데몬이 재시작될 경우 메모리 내 클레임 상태가 유실될 수 있으며, 데이터베이스에 영속화되는지 미확인이다.

## provenance

라우터 파일 `sdi-plugin/crates/daemon/src/router/scenario.rs` 에서 `GET /scenarios/active-claims` 핸들러를 추론하여 작성하였다. D29 파일 클레임 설계, PreToolUse 훅과의 협력 구조, 그리고 claim/release 엔드포인트와 함께 구성되는 클레임 대장 개념에서 도출하였다.

## 미확정 (OPEN)

- 활성 클레임 데이터가 SQLite에 영속화되는지, 아니면 데몬 메모리에만 유지되는지 미확인. 데몬 재시작 시 클레임 상태가 유실될 경우 고아 클레임 없이 자동 초기화된다는 의미이므로 정책 결정 필요.
- 응답 항목에 클레임을 등록한 세션 식별자가 포함되는지 미확인. 포함되지 않으면 어느 세션이 특정 경로를 점유 중인지 추적 불가.
- 파일 경로 글로브 겹침(overlap) 판단을 이 엔드포인트가 직접 수행하는지(입력 파일 경로를 받아 충돌 여부를 반환), 아니면 단순 목록만 반환하고 글로브 매칭은 훅 측에서 하는지 미확인.
- `plan_id` 없이 전체 클레임 대장을 반환할 때 데몬 운영 규모에 따라 항목이 많아질 수 있다. 페이지네이션 지원 여부 미확인.
