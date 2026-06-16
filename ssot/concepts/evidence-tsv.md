---
id: concept.evidence-tsv
kind: Concept
title: Evidence TSV(TSV 증거 형식)
definition: "시나리오 검증 결과를 탭 구분값(TSV) 형식으로 기록하는 표현 방식. TaskEvidence.scenarios 배열의 각 항목이 실질적으로 이 형식의 한 행에 해당한다. scenario_id, result(passing/failing/impacted/retired), evidence_ref의 세 필드로 구성된다."
relatesTo:
  - { to: concept.task-evidence, type: backed-by, note: "Evidence TSV는 TaskEvidence 내 ScenarioEvidence 목록의 구조화 표현이다." }
  - { to: concept.scenario, type: relates-to, note: "TSV의 각 행은 하나의 시나리오에 대한 검증 판정 결과다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/task.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Evidence TSV는 시나리오 검증 결과를 구조화된 형식으로 표현하는 방법이다. SDI에서 태스크를 완료할 때 제출하는 TaskEvidence의 `scenarios` 배열이 사실상 이 TSV 구조를 표현한다.

각 시나리오 증거 항목은 세 가지 정보를 담는다.

1. **scenario_id**: 어떤 시나리오에 대한 판정인지.
2. **result**: 판정 결과 — passing(통과), failing(실패), impacted(이번 변경의 영향을 받음), retired(은퇴 처리됨).
3. **evidence_ref**: 판정 근거를 확인할 수 있는 위치 — "파일:라인", URL, 로그 경로 등. 비어 있으면 안 된다.

추가로 `note` 필드에 자유 서술 메모를 남길 수 있다.

이 구조를 TSV라고 부르는 것은, test-runner specialist가 검증 결과를 탭 구분 텍스트 형태로 정리해 evidence 페이로드를 구성하는 워크플로우에서 유래한 명칭이다.

## 엔티티 (DB)

Evidence TSV는 독립 테이블이 아니다. tasks 테이블의 `evidence` JSON 컬럼 내 `scenarios` 배열 항목으로 저장된다.

## API 표면

태스크 완료 시 evidence 페이로드의 `scenarios` 배열로 전달된다.

## 불변식

- **evidence_ref 비어있으면 안 됨**: 각 항목의 evidence_ref는 공백 제거 후 비어있으면 안 된다.
- **결과값 제한**: result는 passing/failing/impacted/retired 중 하나여야 한다.

## 구현 위치 (provenance)

`ScenarioEvidence` 구조체와 검증 로직은 `sdi-plugin/crates/core/src/task.rs`에 있다.

## 미확정 (OPEN)

- [ ] OPEN: "TSV 형식"이 실제로 CLI나 MCP 도구에서 탭 구분 텍스트로 파싱되는 경로가 있는지, 또는 JSON 직렬화만 사용하는지 확인 필요.
