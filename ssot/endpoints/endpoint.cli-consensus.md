---
id: endpoint.cli-consensus
kind: Endpoint
title: "CLI `sdi consensus`"
definition: "D20 M3 4단계 협의(제안→비평→합의→이견) 현황 조회"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/consensus.rs"
  - "sdi-plugin/plugin/commands/consensus.md"
relatesTo:
  - to: "concept.consensus"
    type: "reads"
  - to: "concept.decision"
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

`sdi consensus` 는 SDI 시스템에서 D20 M3 협의 프로세스의 현황을 조회하는 CLI 명령이다. 다수의 에이전트가 참여하는 의사결정 과정을 4단계 구조(제안→비평→합의→이견)로 가시화하는 읽기 전용 집계 뷰다.

M3 협의 프로세스는 SDI에서 중요한 결정이 단일 에이전트의 판단이 아닌 명시적 합의 절차를 거쳐 도달하도록 설계된 메커니즘이다. `sdi consensus`는 그 과정이 어느 단계에 있는지를 플랜의 결정 레코드를 집계하여 보여준다.

## 요청 / 응답

**현황 조회(view)**: 지정된 플랜의 모든 결정 레코드를 가져와 `proposal_id` 기준으로 묶고, 각 제안의 현재 단계를 분류하여 표시한다.

4단계 분류는 다음과 같다.

- **제안(proposal)**: 초기 제안이 제출되었으나 아직 비평을 받지 않은 상태
- **비평(critique)**: 하나 이상의 비평이 제출된 상태. 찬반 논거가 축적 중
- **합의(consensus)**: 참여 에이전트들이 특정 방향으로 수렴하여 결정이 확정된 상태
- **이견(dissensus)**: 교착 상태 또는 해소되지 않은 이견이 존재하여 에스컬레이션이 필요한 상태

이견(dissensus) 상태에서는 교착 제안 목록과 권장 복구 패턴(에스컬레이션 경로, 대안 제안 제출 방법)을 함께 안내한다.

## 권한 / 제약

- `sdi consensus`는 읽기 전용 집계 뷰다. 결정 데이터를 직접 변경하지 않는다.
- 합의 프로세스의 실제 진행(제안 제출, 비평 작성, 합의 선언)은 `sdi decide` 명령 또는 M3 협의 에이전트를 통해 이루어진다.
- `proposal_id` 기준 집계는 결정 레코드의 메타데이터에 의존한다. 결정이 `proposal_id`없이 기록되어 있으면 개별 독립 결정으로 분류된다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/consensus.rs` 및 `sdi-plugin/plugin/commands/consensus.md` 파일 구조와 SDI PRD D20 M3 협의 설계를 근거로 추론되었다. `proposal_id` 기준 집계 방식은 설계 의도 기반 추론이며 코드 검증 대기 중이다.

## 미확정 (OPEN)

- `sdi consensus`가 단순 조회 외에 이견 에스컬레이션을 직접 트리거하는 기능을 포함하는지 불확실하다.
- `proposal_id` 필드가 `sdi decide create` 시 자동 생성되는지, 사용자가 수동으로 지정해야 하는지 확인이 필요하다.
- 합의 판정 기준(참여 에이전트의 몇 퍼센트가 동의해야 합의로 선언되는지)의 수치 정책이 명확하지 않다.
- `sdi consensus` 명령이 별도 서브커맨드(예: `view`, `list`) 없이 단일 뷰로만 동작하는지, 또는 복수 서브커맨드를 가지는지 확인이 필요하다.
