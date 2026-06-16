---
id: concept.task-evidence
kind: Concept
title: TaskEvidence(태스크 증거)
definition: "태스크가 완료(done)로 전환될 때 반드시 첨부해야 하는 검증 근거 묶음. 시나리오별 판정 결과(Pass/Fail/Impacted/Retired)와 근거 참조(파일:라인, URL, 로그 경로 등)를 담는다. 빈 Evidence로는 done 전환이 불가능하다."
relatesTo:
  - { to: concept.task, type: belongs-to, note: "Evidence는 태스크에 종속되며 태스크의 done 전환 조건이다." }
  - { to: concept.scenario, type: relates-to, note: "Evidence 안의 각 ScenarioEvidence 항목은 해당 시나리오의 ID와 판정 결과를 가리킨다." }
  - { to: concept.round, type: relates-to, note: "Evidence는 라운드별 검증 결과를 기록하는 핵심 데이터다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/task.rs
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

TaskEvidence는 "이 태스크가 정말로 의도된 바를 달성했는가"를 검증하는 근거 다발이다. SDI가 증거를 필수로 요구하는 이유는 "완료했다"는 선언만으로는 라운드의 회귀 추적이 불가능하기 때문이다. 증거가 있어야 다음 라운드에서 "이전 라운드에서 passing이었던 시나리오가 지금도 passing인가"를 기계적으로 확인할 수 있다.

Evidence는 세 부분으로 구성된다.

1. **시나리오별 판정 목록(scenarios)**: 각 항목은 어떤 시나리오에 대해 어떤 결과가 나왔는지(passing/failing/impacted/retired)와, 그 결과를 확인할 수 있는 위치(파일:라인, URL, 로그 경로 등)를 담는다.
2. **요약(summary)**: 자유 서술 요약. 시나리오 목록이 없을 때 최소한 이 필드라도 채워야 done 전환이 허용된다.
3. **추가 항목(extras)**: 문자열 키-값 맵. 자유 확장 공간.

done 전환이 허용되려면 적어도 하나는 채워져야 한다: scenarios 목록이 있거나, 또는 summary가 있어야 한다. 둘 다 비면 `EVIDENCE_REQUIRED` 오류로 거부된다.

시나리오별 판정 결과의 의미는 다음과 같다.

- **passing**: 이 시나리오 기준에 맞게 구현되어 있음.
- **failing**: 이 시나리오 기준을 현재 코드가 충족하지 못함.
- **impacted**: 이번 태스크의 변경이 이 시나리오에 영향을 미쳤음(회귀 신호).
- **retired**: 이 시나리오는 이번 라운드에서 은퇴 처리됨.

## 엔티티 (DB)

TaskEvidence는 tasks 테이블의 evidence 컬럼에 JSON으로 직렬화되어 저장된다. 별도 테이블이 아닌 inline JSON 필드다.

## API 표면

태스크 완료 API(`POST /tasks/:id/complete` 혹은 해당 MCP 도구)를 호출할 때 evidence 페이로드를 함께 전달한다. 빈 Evidence(`{}`) 또는 scenarios·summary 둘 다 비어 있는 Evidence는 API 계층에서 즉시 거부된다.

## 불변식

- **done에 Evidence 필수**: done 전환 시 scenarios와 summary 둘 다 비어 있으면 거부.
- **evidence_ref 비어있으면 안 됨**: scenarios 배열 각 항목의 evidence_ref는 공백 후 비어서는 안 된다.

## 구현 위치 (provenance)

`TaskEvidence` 구조체 정의 및 `validate_for_done` 로직은 `sdi-plugin/crates/core/src/task.rs`에 있다. `ScenarioEvidence`도 같은 파일에 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: evidence의 크기 상한 — Clawket에서는 4KiB 상한이 API 레이어에서 강제되었는데 SDI에서도 동일 정책이 적용되는지 확인 필요.
