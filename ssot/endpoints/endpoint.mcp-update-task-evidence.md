---
id: endpoint.mcp-update-task-evidence
kind: Endpoint
title: MCP update_task_evidence — 태스크 검증 증거 기록 및 완료
definition: LLM 에이전트가 태스크 완료 시 시나리오별 검증 결과와 증거 참조를 기록하고 태스크를 done으로 전환하는 MCP 쓰기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/write.rs
relatesTo:
  - to: concept.task-evidence
    type: mutates
  - to: concept.task
    type: mutates
  - to: concept.scenario
    type: reads
  - to: domain.task-decomposition
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

`update_task_evidence` 는 SDI MCP 서버의 쓰기 도구로, LLM 에이전트가 태스크 구현을 마쳤을 때 "무엇을 어떻게 검증했는가"를 기록하고 태스크를 완료 상태로 전환하는 도구다. 데몬의 `POST /tasks/:id/complete` 엔드포인트로 위임된다.

SDI 에서 태스크 완료는 단순히 상태를 `done`으로 바꾸는 것이 아니다. 태스크가 책임지고 있는 각 시나리오에 대해 실제로 검증이 이루어졌는지, 그 결과가 어떠한지, 어떤 커밋이나 파일이 그 근거인지를 함께 제출해야 한다. 이 증거 기록이 SDI 전체 감사 추적의 핵심 고리다.

이 도구는 또한 D6 회귀 정책과 직접 연결된다. 증거 배열에 포함된 시나리오의 결과가 `failing`이나 `impacted`로 기록되면, 데몬이 회귀 감지를 발동하여 해당 라운드에 이월된 베이스라인과 비교한다.

## 요청 / 응답

**입력이 의미하는 것**

태스크 ID는 어느 태스크를 완료 처리할지를 지정한다. 그 다음에 오는 증거 배열이 이 도구의 핵심이다. 배열의 각 항목은 세 가지 정보를 담는다.

첫째는 시나리오 ID로, 이 태스크가 어떤 시나리오를 검증했는지를 명시한다. 둘째는 결과로, 네 가지 중 하나를 선택한다: `passing`(시나리오가 의도한 대로 동작함이 확인됨), `failing`(기대 동작이 충족되지 않음), `impacted`(코드 변경으로 인해 이 시나리오의 동작이 영향을 받았으나 결과가 아직 불확실함), `retired`(시나리오가 더 이상 적용 가능하지 않아 폐기됨). 셋째는 증거 참조로, 이 판정의 근거가 되는 구체적인 산출물을 가리킨다 — 커밋 해시, 파일 경로, 테스트 실행 결과 로그 경로 등이 해당한다.

**출력이 의미하는 것**

호출이 성공하면 완료 처리된 태스크 정보와 실제로 기록된 증거 건수를 반환한다. 태스크 상태는 `done`으로 전환된다. 데몬은 기록된 증거를 바탕으로 라운드 진행 상태와 회귀 베이스라인 비교를 업데이트한다.

## 권한 / 제약

증거 배열은 비어 있으면 안 된다. 빈 배열로 호출하면 MCP 레이어 자체에서 데몬 호출 없이 즉시 거부한다. 태스크가 시나리오와 연결되어 있다면 반드시 그에 대한 검증 결과를 함께 제출해야 한다.

각 시나리오 ID는 해당 태스크와 연결된 플랜 내에 실제로 존재해야 한다. 존재하지 않는 시나리오 ID를 지정하면 데몬이 거부한다.

이 도구는 태스크 상태를 `done`으로 전환하므로, 이후 추가 증거를 기록하거나 결과를 수정하는 것이 가능한지는 미확인이다. 완료 전 충분한 검증이 선행되어야 한다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/write.rs` 구현 분석에서 추론
- 위임 엔드포인트: `POST /tasks/:id/complete` (데몬 REST API)
- 관련 설계 결정: D6 (회귀 정책 — 증거 기반 라운드 진행), 태스크-시나리오 연결 모델

## 미확정 (OPEN)

- 증거 배열의 최대 항목 수 제한이 있는지 불명확
- `done` 처리 이후 증거를 추가하거나 결과를 수정하는 경로가 존재하는지 미확인
- `impacted` 결과의 후속 처리 흐름(D9 disruption-review 발동 여부)이 코드로 확인 필요
- 태스크가 시나리오와 연결되어 있지 않은 경우(연결 없는 단순 작업 태스크)에도 이 도구를 통해 완료 처리하는지, 다른 경로가 있는지 미확인
