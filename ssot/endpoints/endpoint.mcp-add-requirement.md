---
id: endpoint.mcp-add-requirement
kind: Endpoint
title: MCP add_requirement — 요구사항 스냅샷 추가
definition: LLM 에이전트가 활성 플랜에 현재 시점의 요구사항을 스냅샷으로 기록하는 MCP 쓰기 도구
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/write.rs
relatesTo:
  - to: concept.requirement
    type: mutates
  - to: concept.plan
    type: reads
  - to: domain.requirements
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

`add_requirement` 는 SDI MCP 서버가 노출하는 쓰기 도구로, LLM 에이전트가 플랜에 요구사항을 기록할 때 사용한다. 내부적으로 SDI 데몬의 `POST /plans/:id/requirements` 엔드포인트로 위임된다.

SDI 에서 요구사항은 D12 스냅샷 의미론을 따른다. 이는 요구사항 문서가 "현재 우리가 무엇을 만들려고 하는가"의 단일 시점 진실을 담는다는 뜻이다. 변경이 생기면 이전 버전을 수정하는 것이 아니라 현재 상태를 덮어써서 최신 진실만 남긴다. 변경 이력이 필요하다면 별도의 결정 로그나 ADR에 기록한다.

## 요청 / 응답

**입력이 의미하는 것**

에이전트는 네 가지 정보를 전달한다. 첫째는 대상 플랜 ID로, 어느 계획에 이 요구사항이 속하는지를 지정한다. 둘째는 요구사항 본문으로, 시스템이 반드시 충족해야 하는 동작이나 품질 속성을 자연어로 기술한다. 기능 요구사항("사용자는 시나리오를 검색할 수 있어야 한다")과 비기능 요구사항("응답 시간은 500ms 이내여야 한다") 모두 이 형식으로 기록한다. 셋째는 우선순위로, 구현 순서와 자원 배분 판단에 사용된다. 넷째는 출처로, 이 요구사항이 어느 이해관계자의 의견에서 비롯되었는지 또는 어느 문서에 근거하는지를 명시한다.

**출력이 의미하는 것**

호출이 성공하면 생성된 요구사항 ID를 반환한다. 이 ID는 이후 시나리오와 요구사항 간 연결을 추적할 때 참조점이 된다.

## 권한 / 제약

대상 플랜이 데이터베이스에 존재해야 한다. 플랜의 상태 조건(active 여부)이 요구사항 추가에도 동일하게 적용되는지는 데몬 구현에 따라 다를 수 있으며 현재 미확인이다.

D12 스냅샷 의미론에 따라, 같은 주제의 요구사항이 변경되었을 때 기존 항목을 업데이트하는 것이 아니라 내용을 현재 상태로 덮어쓰는 방식이 올바른 사용 패턴이다. 한 플랜 내에 같은 ID를 가리키는 요구사항이 이력 형태로 누적되어서는 안 된다. 변경 이력은 이 도구가 아닌 `add_decision` 도구를 통해 ADR로 별도 기록한다.

MCP 서버 전체 공통 제약으로, SDI 데몬(`sdid`)이 실행 중이어야 하며 포트 파일(`~/.cache/sdi/sdid.port`)이 존재해야 한다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/write.rs` 구현 분석에서 추론
- 위임 엔드포인트: `POST /plans/:id/requirements` (데몬 REST API)
- 관련 설계 결정: D12 (스냅샷 의미론 — 문서는 현재 시점 단일 진실)

## 미확정 (OPEN)

- 요구사항 추가 시 플랜 상태(active 여부) 제약이 명시적으로 강제되는지 미확인
- 우선순위 값의 허용 범위(숫자 척도인지, 레이블인지: high/medium/low 등)가 불명확
- 같은 출처·내용의 요구사항 중복 등록에 대한 중복 감지 여부 미확인
- 요구사항과 시나리오 간 연결 관계가 이 도구 호출 시점에 설정 가능한지 미확인
