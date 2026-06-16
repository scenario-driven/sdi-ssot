---
id: endpoint.cli-decide
kind: Endpoint
title: "CLI `sdi decide`"
definition: "D12 Append-only ADR 로그 관리 — 결정 생성·조회·대체·롤백"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/decision.rs"
  - "sdi-plugin/plugin/commands/decide.md"
relatesTo:
  - to: "concept.decision"
    type: "mutates"
  - to: "concept.plan"
    type: "reads"
  - to: "domain.decision-negotiation"
    type: "belongs-to"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`sdi decide` 는 SDI 시스템에서 아키텍처 결정 기록(ADR, Architecture Decision Record)을 관리하는 CLI 명령 그룹이다. 시나리오가 "어떻게 동작해야 하는가"를, 요구사항이 "무엇을 만들어야 하는가"를 담는다면, 결정은 "왜 그렇게 결정했는가"의 근거와 맥락을 영구 보존하는 객체다.

D12 Append-only 원칙이 결정 관리의 핵심 제약이다. 한 번 기록된 결정은 직접 수정할 수 없다. 결정이 번복되거나 갱신되었을 때는 `supersede` 명령으로 이전 결정을 대체하는 새 결정을 체이닝 방식으로 연결한다. 이 방식은 의사결정 과정의 진화를 단일 연결 체인으로 추적 가능하게 만든다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: 플랜에 귀속된 새 결정 레코드를 생성한다. 결정 제목, 문맥(Context), 선택한 방향(Decision), 근거(Rationale), 영향(Consequences)을 입력받는다. 생성 즉시 확정 상태로 등록된다.

**조회(list/view)**: 플랜 범위 내 결정 목록을 시간 역순으로 반환한다. `view`는 단일 결정의 전체 내용과 대체 체인(superseded_by, supersedes) 정보를 함께 표시한다.

**대체(supersede)**: 기존 결정을 폐기하고 그것을 대체하는 새 결정을 생성한다. 이전 결정은 `superseded` 상태로 전환되며, 새 결정에 이전 결정의 ID가 `supersedes` 필드로 연결된다. 결정 히스토리 체인이 유지된다.

**롤백(rollback)**: 특정 결정을 실행 취소하는 역전 계획(rollback plan)을 읽어 처리한다. 롤백 계획은 JSON 직접 입력 또는 파일 경로로 지정한다. 롤백은 해당 결정에 연결된 구현 변경을 되돌리는 역보상 액션을 포함할 수 있다.

## 권한 / 제약

- Append-only: 기존 결정의 `title`, `context`, `decision`, `rationale`, `consequences` 필드는 직접 수정 불가하다. 번경이 필요하면 반드시 `supersede`를 사용해야 한다.
- `rollback`은 역전 계획이 명시적으로 정의되어야만 실행된다. 역전 계획 없는 단순 결정 삭제는 허용되지 않는다.
- `supersede`로 대체된 결정은 조회는 가능하나 활성 상태로 되돌릴 수 없다.
- 결정은 반드시 활성 플랜에 귀속되어야 한다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/decision.rs` 및 `sdi-plugin/plugin/commands/decide.md` 파일 구조와 SDI PRD D12 Append-only 설계 규칙을 근거로 추론되었다. `rollback` 서브커맨드와 역전 계획 형식은 SDI 설계 문서 기반 추론이며, 코드 구현 세부 사항은 별도 검증이 필요하다.

## 미확정 (OPEN)

- `rollback` 명령이 자동으로 역보상 액션을 실행하는지, 아니면 계획만 기록하고 실행은 사람이 하는지 명확하지 않다.
- 결정에 첨부 가능한 메타데이터(예: 제안자, 합의 참여자, 합의 단계 연결)의 스펙이 확인되지 않았다.
- `supersede` 체인의 깊이 제한 여부(무한 체이닝 허용 여부)가 불확실하다.
