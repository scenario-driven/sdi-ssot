---
id: endpoint.cli-plan
kind: Endpoint
title: "CLI `sdi plan`"
definition: "D1/D8 플랜 생애주기 관리 — 생성·승인·완료·활성 플랜 조회·컨텍스트 스냅샷"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/plan.rs"
  - "sdi-plugin/plugin/commands/plan.md"
relatesTo:
  - to: "concept.plan"
    type: "mutates"
  - to: "concept.scenario"
    type: "reads"
  - to: "concept.task"
    type: "reads"
  - to: "concept.decision"
    type: "reads"
  - to: "domain.planning"
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

`sdi plan` 은 SDI 시스템에서 구현 계획 단위인 플랜(Plan)의 생애주기를 관리하는 CLI 명령 그룹이다. 플랜은 시나리오·요구사항·태스크·결정을 하나의 목적 아래 묶는 최상위 작업 컨테이너다. D1 규칙에 따라 모든 시나리오와 태스크는 반드시 플랜에 귀속된다.

D8 승인 게이트가 플랜 생애주기의 핵심 제약이다. 플랜은 `draft → active → completed` 순으로만 전이하며, `approve`(draft→active 전환)를 거치지 않은 플랜에서는 어떤 태스크도 시작할 수 없다. 이 게이트는 미성숙한 계획이 구현 단계로 넘어가는 것을 방지하는 통제 장치다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**생성(create)**: 플랜 제목과 설명을 입력받아 draft 상태의 플랜을 생성한다. 생성 시 프로젝트에 귀속되며, 초기에는 시나리오·태스크·결정이 비어 있다.

**조회(list/view)**: 프로젝트 범위 내 전체 플랜 목록을 상태별로 필터링하여 반환한다. `view`는 단일 플랜의 세부 내용과 귀속된 시나리오 수, 태스크 진행 현황을 함께 표시한다.

**수정(update)**: 플랜 제목·설명·메타데이터를 수정한다. D12 스냅샷 원칙에 따라 현재 값만 유지한다.

**승인(approve)**: draft 상태의 플랜을 active 상태로 전환하는 D8 게이트 명령이다. 이 명령 이후에만 플랜 내 태스크 시작이 허용된다.

**완료(complete)**: active 상태의 플랜을 completed 상태로 전환한다. 진행 중인 태스크가 남아 있을 경우 경고가 표시된다.

**활성 플랜 조회(active)**: 현재 cwd(작업 디렉토리)의 프로젝트에서 active 상태인 플랜을 빠르게 조회한다. 에이전트가 작업 컨텍스트를 파악하는 용도로 자주 사용된다.

**컨텍스트 스냅샷(context)**: 플랜에 귀속된 시나리오 목록, 진행 중 태스크, 최근 결정을 단일 스냅샷으로 조합하여 반환한다. MCP `get-plan-context` 도구와 동일한 데이터를 CLI에서 확인하는 용도다.

## 권한 / 제약

- D8 게이트: `approve` 없이는 플랜 내 태스크를 시작할 수 없다. 이 제약은 데몬 레벨에서 강제된다.
- 플랜 상태 전이는 단방향이다. completed 상태에서 active로 되돌릴 수 없다.
- `complete` 시 미완료 태스크가 있으면 경고가 발생하나, 강제 완료는 허용될 수 있다(정책 미확정).
- D12 스냅샷 원칙 적용: 플랜 본문에 변경 이력을 남기지 않는다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/plan.rs` 및 `sdi-plugin/plugin/commands/plan.md` 파일 구조와 SDI PRD D1/D8 설계 규칙을 근거로 추론되었다. `context` 서브커맨드와 MCP `get-plan-context`의 데이터 동일성은 설계 의도 기반 추론이며 코드 검증 대기 중이다.

## 미확정 (OPEN)

- `complete` 명령이 미완료 태스크가 있을 때 강제 완료를 허용하는지 차단하는지 확인이 필요하다.
- `approve` 권한이 특정 에이전트 역할(예: architect)에만 제한되는지, 아니면 누구나 실행 가능한지 정책이 명확하지 않다.
- `context` 서브커맨드의 출력 형식(JSON vs. 사람 가독 텍스트)과 MCP 응답과의 정확한 일치 여부가 확인되지 않았다.
