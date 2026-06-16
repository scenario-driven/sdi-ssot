---
id: endpoint.cli-agent-note
kind: Endpoint
title: "CLI `sdi agent-note`"
definition: "M1 블랙보드 에이전트 노트 CRUD + M2 핸드오프 수신·확인·은퇴"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/agent_note.rs"
  - "sdi-plugin/plugin/commands/agent-note.md"
relatesTo:
  - to: "concept.agent-note"
    type: "mutates"
  - to: "domain.agent-coordination"
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

`sdi agent-note` 는 SDI 시스템에서 에이전트 간 비동기 정보 공유와 작업 인수인계를 지원하는 에이전트 노트(AgentNote) 객체를 관리하는 CLI 명령 그룹이다.

M1 블랙보드 패턴을 구현한다. 에이전트 노트는 특정 에이전트가 자신의 관찰, 지시사항, 요약, 또는 다음 에이전트에게 전달할 정보를 구조화된 형태로 공유 공간(블랙보드)에 기록하는 메커니즘이다. M2 핸드오프 패턴에서는 핸드오프 종류의 노트를 통해 한 에이전트에서 다른 에이전트로 작업이 이전된다.

에이전트 노트는 플랜, 태스크, 또는 라운드에 앵커링되어 해당 작업 맥락 내에서 공유된다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**노트 추가(append)**: 새 에이전트 노트를 생성한다. 노트 종류(kind)는 observation(관찰 결과), handoff(작업 인수인계), directive(지시사항), summary(작업 요약) 중 하나를 선택한다. 노트가 귀속될 스코프(플랜 ID, 태스크 ID, 라운드 ID)와 수신 대상 에이전트를 함께 지정한다.

**노트 목록 조회(list)**: 지정된 스코프 내 에이전트 노트 목록을 종류·상태·작성 에이전트 기준으로 필터링하여 반환한다.

**핸드오프 수신함 조회(handoffs)**: 현재 세션 에이전트에게 보내진 미확인(unacknowledged) 핸드오프 노트 목록을 반환한다. 에이전트가 새 작업을 시작하거나 세션을 재개할 때 우선 확인해야 할 인수인계 항목을 파악하는 용도로 사용된다.

**수신 확인(ack)**: 핸드오프 노트를 수신 확인(acknowledged) 상태로 전환한다. 수신 확인 후 해당 노트는 `handoffs` 목록에서 제거된다.

**노트 은퇴(retire)**: 더 이상 유효하지 않은 노트를 비활성화 상태로 전환한다. 은퇴된 노트는 기본 목록 조회에서 제외되나, 이력 조회는 유지된다.

## 권한 / 제약

- 에이전트 노트는 반드시 플랜, 태스크, 라운드 중 하나 이상의 스코프에 앵커링되어야 한다. 스코프 없는 부유(free-floating) 노트는 허용되지 않는다.
- `handoffs` 조회는 세션 에이전트 신원에 의존하므로, 에이전트 식별자가 명확하게 설정되어 있어야 한다.
- `ack` 없이 핸드오프된 작업을 시작하는 것이 가능한지, 아니면 수신 확인이 선행 조건인지 정책이 별도로 존재할 수 있다.
- `retire`된 노트는 복원(unretire) 가능 여부가 명확하지 않다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/agent_note.rs` 및 `sdi-plugin/plugin/commands/agent-note.md` 파일 구조와 SDI M1(블랙보드), M2(핸드오프) 설계 패턴을 근거로 추론되었다. 노트 종류(observation/handoff/directive/summary) 목록은 설계 명세 기반 추론이다.

## 미확정 (OPEN)

- 에이전트 신원 식별 방식(세션 에이전트 ID의 출처와 설정 방법)이 코드 레벨에서 확인되지 않았다.
- `handoffs` 목록에서 미확인 핸드오프가 오랫동안 방치될 경우 자동 알림이나 에스컬레이션 정책이 있는지 불확실하다.
- `retire`된 노트의 복원 가능 여부와 완전 삭제 지원 여부가 명확하지 않다.
- directive 종류 노트가 단순 기록인지, 아니면 수신 에이전트의 행동에 구속력을 갖는지 의미론적 구분이 명확하지 않다.
