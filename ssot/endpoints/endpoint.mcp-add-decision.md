---
id: endpoint.mcp-add-decision
kind: Endpoint
title: MCP add_decision — ADR append-only 기록
definition: LLM 에이전트가 플랜에 의사결정 기록(ADR)을 추가하는 MCP 쓰기 도구로, 기존 결정을 supersedes 체이닝으로 갱신한다
realizedBy: []
implementedIn:
  - sdi-plugin/crates/mcp/src/tools/write.rs
relatesTo:
  - to: concept.decision
    type: mutates
  - to: concept.plan
    type: reads
  - to: domain.decision-negotiation
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

`add_decision` 은 SDI MCP 서버의 쓰기 도구로, LLM 에이전트나 M3 협상 참여자가 특정 플랜에 의사결정 기록(Architecture Decision Record)을 남길 때 호출한다. 데몬의 `POST /plans/:id/decisions` 엔드포인트로 위임된다.

SDI 에서 결정은 append-only 원칙으로 관리된다. 한 번 기록된 결정의 본문은 직접 수정할 수 없으며, 결정을 번복하거나 보완할 때는 새 결정을 추가하면서 기존 결정 ID를 `supersedes_id`로 지정하는 체이닝 방식을 사용한다. 이 체인이 형성되면 데몬은 참조된 기존 결정을 자동으로 `superseded` 상태로 전환한다. 이 구조 덕분에 결정 이력이 명시적으로 보존되고, "이 결정이 왜 바뀌었는가"를 나중에 추적할 수 있다.

## 요청 / 응답

**입력이 의미하는 것**

에이전트는 다섯 가지 정보를 전달한다. 첫째는 플랜 ID로, 이 결정이 어느 계획 범위에 속하는지를 지정한다. 둘째는 결정 제목으로, 한 줄로 읽어도 결정의 핵심이 파악되는 이름을 쓴다. 셋째는 결정 내용으로, 배경(왜 이 문제를 결정해야 했는가), 검토한 선택지(어떤 대안들이 있었는가), 선택 근거(왜 이 방향을 골랐는가)를 담는다. 넷째는 선택적 `supersedes_id`로, 이 결정이 기존의 어느 결정을 대체하는지를 지정한다. 다섯째는 선택적 상태로, `provisional`(잠정적 — 추후 재검토 예정)이나 `confirmed`(확정)를 지정할 수 있다.

**출력이 의미하는 것**

호출이 성공하면 새로 생성된 결정 ID를 반환한다. `supersedes_id`를 지정한 경우에는 참조된 기존 결정이 `superseded` 상태로 전환되었음도 응답에 포함되어 체인 형성을 확인할 수 있다.

## 권한 / 제약

결정 본문의 직접 수정은 불가하다. 잘못 작성된 결정이 있더라도 해당 결정을 삭제하거나 편집하는 것이 아니라, 수정된 내용을 담은 새 결정을 `supersedes_id`로 연결하는 방식으로만 갱신한다.

`supersedes_id`를 지정하는 경우 참조 대상 결정이 같은 플랜 내에 존재해야 한다. 다른 플랜의 결정을 참조하는 교차 플랜 체이닝은 데몬에서 거부된다.

대상 플랜이 존재해야 하며, 플랜 상태 조건이 적용될 수 있다(미확인). M3 4단계 협상 프로세스(제안 → 비판 → 합성 → 확정)와 연계하여 사용하는 경우, 협상 완료 후 `confirmed` 상태로 등록하는 흐름을 권장한다.

## provenance

- 출처: `sdi-plugin/crates/mcp/src/tools/write.rs` 구현 분석에서 추론
- 위임 엔드포인트: `POST /plans/:id/decisions` (데몬 REST API)
- 관련 설계 결정: M3 (4단계 협상 프로세스), append-only 이력 보존 원칙

## 미확정 (OPEN)

- `supersedes_id` 지정 시 기존 결정의 상태 전환(`superseded`)이 데몬에서 원자적으로 처리되는지, 아니면 별도 단계로 처리되는지 미확인
- `provisional` 상태 결정이 자동으로 `confirmed`로 승격되는 조건(타임아웃, 반론 없음 등)이 있는지 미확인
- 플랜 상태(active 여부)가 결정 추가의 전제 조건인지 미확인
- M3 협상 결과 외에 단독으로 호출될 때의 거버넌스 규칙(누가 호출할 수 있는지)이 불명확
