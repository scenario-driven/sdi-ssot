---
id: invariant.decision-append-only
kind: Invariant
title: Decision은 append-only ADR이다 — 기존 행을 수정하거나 삭제할 수 없다
definition: "Decision 테이블의 행은 생성(INSERT)만 가능하며, 기존 행의 수정(UPDATE) 또는 삭제(DELETE)는 허용되지 않는다. proposal → critique → consensus/dissensus의 합의 단계는 새 행을 추가하는 방식으로 표현된다. rollback도 원 Decision을 수정하는 것이 아니라 새 Decision 행(reversal_of 참조)을 추가하는 방식으로 처리된다."
governs:
  - concept.decision
  - concept.consensus
  - domain.decision-negotiation
  - domain.governance-audit
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/decision.rs"
  - "sdi-plugin/crates/core/src/decision.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

SDI의 Decision은 구현 선택, 아키텍처 결정, 네이밍, 스키마 변경 등 중요한 판단을 기록하는 Architectural Decision Record(ADR)다. 한번 기록된 Decision 행은 변경되지 않는다. 의사결정의 진행은 새 행을 추가하는 방식으로 이루어진다.

합의 단계가 append-only로 표현된다. 어떤 에이전트가 제안(`proposal`)을 내면 새 Decision 행이 생성된다. 다른 에이전트가 비판(`critique`)을 내면 또 다른 행이 추가된다. 합의에 도달하면 `consensus` 행이, 충돌이 발생하면 `dissensus` 행이 추가된다. 이 흐름에서 어떤 행도 수정되거나 삭제되지 않는다.

D28의 rollback도 같은 원칙을 따른다. 잘못된 Decision을 되돌리려면 원 Decision 행을 수정하는 것이 아니라 `kind='consensus', reversal_of=<원 id>`인 새 Decision 행을 추가한다. 원 행은 영구적으로 보존된다.

이 원칙의 이유는 감사 가능성(auditability)이다. "누가, 언제, 어떤 근거로 어떤 결정을 했는가"의 전체 히스토리가 append-only 스트림에 보존된다. 수정이나 삭제가 허용되면 과거 판단이 재작성될 수 있어 감사 추적의 신뢰성이 무너진다.

## 깨지면 무슨 일이 일어나나

Decision 행이 수정 또는 삭제될 수 있다면 의사결정 이력의 신뢰성이 사라진다. "왜 이 아키텍처를 선택했는가"를 나중에 재구성할 때 일부 판단이 이미 조용히 수정된 상태일 수 있다. Rollback을 원 행의 수정으로 처리하면 "이 결정이 실제로 적용됐다가 번복됐다"는 사실이 보이지 않아진다. M3 협상 흐름(proposal → critique → consensus)도 중간 단계 행이 삭제되면 어떤 과정을 거쳐 합의에 도달했는지 재현 불가능해진다. L5 자율 적용의 근거인 "이 결정의 reversal_plan이 존재한다"도 원 행이 수정되면 검증할 수 없다.

## 코드에서 어떻게 강제되나

데몬의 Decision 라우터(`crates/daemon/src/router/decision.rs`)는 `POST /decisions` (INSERT)만 노출한다. `PATCH /decisions/:id`나 `DELETE /decisions/:id` 엔드포인트가 존재하지 않는다. Rollback 경로는 `POST /decisions/:id/rollback`이며, 이 엔드포인트는 원 행을 건드리지 않고 `reversal_of` FK를 가진 새 행을 INSERT한다. D12 SNAPSHOT-ONLY 원칙의 Decision 전용 구현이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(D12 Decision append-only 정책의 결정 엔트리) 연결 필요
