---
id: invariant.evidence-required-on-done
kind: Invariant
title: Task를 done으로 전환할 때는 구조화된 증거가 필수다
definition: "Task의 상태를 done으로 전환하려면, 연결된 시나리오 각각에 대해 scenario_id + result(passing/failing) + 비어 있지 않은 evidence_ref를 포함한 구조화 증거를 반드시 제출해야 한다. 자유 형식 문자열 증거는 허용되지 않는다."
governs:
  - concept.task
  - concept.task-evidence
  - concept.evidence-tsv
  - concept.scenario-claim
  - domain.task-decomposition
  - domain.round-execution
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/task.rs"
  - "sdi-plugin/crates/core/src/task.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

Task의 `status`가 `done`이 되려면 반드시 `TaskEvidence` 구조체를 포함한 완료 요청을 제출해야 한다. 이 구조체의 `scenarios` 배열은 Task가 충족하려는 시나리오 각각에 대해 다음 세 가지를 요구한다: (1) 어떤 시나리오인지를 나타내는 `scenario_id`, (2) 해당 시나리오가 통과했는지 여부를 나타내는 `result`, (3) 그 결론의 근거가 되는 비어 있지 않은 `evidence_ref`.

증거는 단순한 메모가 아니라 Round의 `scenario_results` 테이블에 직접 기입되는 데이터다. 즉, Task가 완료될 때마다 해당 Round에서 어떤 시나리오가 어떤 상태인지가 자동으로 갱신된다. 이 갱신이 R2+ 회귀 검증의 원본 데이터 소스다.

자유 형식 문자열은 이 흐름에서 원천 차단된다. "완료됐음", "테스트 통과" 같은 서술은 어떤 시나리오에 대한 것인지, 어떤 근거로 passing인지 기계적으로 추적할 수 없기 때문이다.

## 깨지면 무슨 일이 일어나나

증거 없이 Task가 done이 되면 Round의 scenario_results가 갱신되지 않는다. R2+ 라운드를 시작할 때 regression-runner는 이전 라운드의 passing 시나리오를 재검증 대상으로 선정하는데, scenario_results가 비어 있거나 불완전하면 어떤 시나리오를 재검증해야 하는지 결정할 수 없다. 결과적으로 strict-regression 모드가 의미를 잃는다. 또한 특정 코드 변경이 어떤 시나리오를 만족시키기 위한 것이었는지 추적이 불가능해져, "이 구현이 왜 이 모양인가"를 다시 파악하려면 git blame과 자연어 해석에만 의존해야 한다.

## 코드에서 어떻게 강제되나

데몬의 태스크 완료 엔드포인트(`crates/daemon/src/router/task.rs`)는 `PATCH /tasks/:id` 요청에서 `status=done` 전환을 시도할 때 `TaskEvidence` 페이로드를 검사한다. `scenarios` 배열이 없거나 비어 있거나 `evidence_ref`가 빈 문자열이면 `EVIDENCE_REQUIRED` 오류를 반환한다. 통과 시 각 항목은 `round_repo::upsert_result`를 통해 `round.scenario_results`에 미러링된다. HOOK_ENFORCEMENT.md #6 인수기준으로도 명시되어 있다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(구조화 증거 필수 정책의 SDI 측 결정 엔트리) 연결 필요
