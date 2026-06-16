---
id: endpoint.projects-next
kind: Endpoint
title: "GET /projects/:id/next"
definition: 프로젝트의 활성 계획 상태를 분석하여, LLM 이 다음에 실행해야 할 하나의 기계적 CLI 명령과 그 이유를 계산해 반환하는 엔드포인트. 결정 상태 머신을 내장하고 있으며 잠정 결정 목록도 함께 제공한다.
realizedBy: []
implementedIn:
  - sdi-plugin/crates/daemon/src/router/aggregate.rs
relatesTo:
  - to: concept.plan
    type: reads
    note: ""
  - to: concept.round
    type: reads
    note: ""
  - to: concept.task
    type: reads
    note: ""
  - to: concept.scenario
    type: reads
    note: ""
  - to: concept.decision
    type: reads
    note: "#16: 잠정(provisional) 결정이 재검토 알림으로 포함됨"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

프로젝트의 현재 상태를 내장 상태 머신으로 평가하여, LLM 이 다음에 실행해야 할 하나의 CLI 명령과 그 이유 문자열을 계산해 반환한다. 상태 머신은 다음 순서로 동작한다.

1. 활성 계획 없음 → 계획 생성 명령 반환
2. 확인(confirmed)된 시나리오 없음 → 시나리오 생성 및 확인 명령 반환
3. 활성 라운드 없음 → 라운드 생성 및 활성화 명령 반환
4. 진행 중(in-flight)인 태스크 존재 → 태스크 브리핑 및 완료 명령 반환
5. 검증이 필요한 시나리오 존재 → 태스크 분해 명령 반환
6. 위 모든 조건에 해당 없음 → 라운드 완료 명령 반환

또한 세션이 계속 인지하고 있어야 할 잠정 결정(provisional decisions, 결정 #16) 목록도 함께 반환한다.

## 요청 / 응답

**GET /projects/:id/next**
- 경로 파라미터: `id`(프로젝트 ID, 필수)
- 응답:
  - `command`: 다음에 실행할 CLI 명령 문자열
  - `reason`: 해당 명령을 제안하는 이유 설명
  - `provisional_decisions`: 세션이 재검토해야 할 잠정 결정 배열

## 권한 / 제약

- 존재하지 않는 프로젝트 ID 를 지정하면 404 를 반환한다.
- 반환된 명령은 기계적으로 계산된 제안이며, LLM 이 이를 따를 의무는 없지만 SDI 워크플로우에서는 이 제안을 기본 진행 방향으로 삼는다.
- 이 엔드포인트는 읽기 전용이며 어떤 상태도 변경하지 않는다.

## provenance

aggregate.rs 라우터에서 구현된다는 정보와 상태 머신의 6단계 구조는 기능 명세 분석을 통해 추론되었다.

## 미확정 (OPEN)

- "검증이 필요한 시나리오" 를 판단하는 기준(예: 태스크가 없는 시나리오인지, 특정 상태인지) 이 명세에 구체적으로 기술되어 있지 않음.
- 잠정 결정(provisional decisions)의 범위 기준이 명확하지 않음(전체 프로젝트 vs 활성 계획 스코프).
- 반환하는 `command` 가 실제 실행 가능한 shell 명령인지, 아니면 `sdi` CLI 서브명령 형태인지 미확인.
