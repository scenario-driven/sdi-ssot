---
id: capability.exchange-agent-notes
kind: Capability
title: 에이전트 간 노트 교환(블랙보드 + 핸드오프)
purpose: "다중 에이전트 흐름에서 전문가 에이전트들이 관찰·질문·경고·요약을 공유 블랙보드에 게시하고, 핸드오프로 다음 에이전트에게 제어를 이전한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/commands/agent-note.md
  - sdi-plugin/plugin/commands/agent-note.md
relatesTo:
  - { to: concept.agent-note, type: mutates, note: "관찰·질문·경고·요약·핸드오프 노트를 생성하고 조회·ack·retire 한다" }
  - { to: concept.handoff, type: mutates, note: "kind=handoff 노트는 to_agent 를 지정하고 ack 가 와야 대기 큐에서 제거된다(M2)" }
  - { to: concept.round, type: reads, note: "노트는 plan/round/scenario/task/global 앵커에 귀속된다" }
  - { to: concept.autonomy-policy, type: relates-to, note: "통신 기판(M1~M5)은 자율성 모드와 독립적으로 동작한다(D19). L3에서도 에이전트 통신은 차단되지 않는다" }
  - { to: concept.collaboration-pattern, type: reads, note: "패턴 구조(workflow/swarm/graph)가 어떤 에이전트끼리 노트를 교환할지를 정한다" }
impacts:
  - concept.agent-note
  - concept.handoff
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

여러 에이전트가 함께 일할 때 서로의 결과물을 채팅 메시지나 툴 출력으로 직접 연결하려 하면 금세 복잡해진다. SDI는 공유 블랙보드 방식을 쓴다. 각 에이전트는 자신이 발견한 사실이나 다음 에이전트에게 전할 정보를 특정 앵커(플랜·라운드·시나리오·태스크)에 노트로 붙인다. 다른 에이전트는 그 앵커의 노트를 읽어 맥락을 이어받는다.

핸드오프(handoff)는 그 중 특별한 종류다. `to_agent`를 지정해 "다음은 네 차례다"라고 명시적으로 전달한다. 수신 에이전트는 자신 앞으로 온 핸드오프를 `handoffs` 명령으로 조회하고, 작업 후 `ack`으로 수신 확인한다. 핸드오프가 체인 구성보다 훨씬 안전하고 추적 가능하다.

## 행위

- `sdi agent-note append <PRJ-ID> --scope plan --plan-id <ID> --kind observation --from impl-coder "<body>"` 로 블랙보드 관찰 게시.
- `sdi agent-note append … --kind handoff --from impl-coder --to test-runner "<body>"` 로 핸드오프 전달.
- `sdi agent-note handoffs <AGENT-NAME>` 으로 자신 앞으로 온 핸드오프 조회.
- `sdi agent-note ack <NOTE-ID>` 로 핸드오프 수신 확인.
- `sdi agent-note retire <NOTE-ID> --reason "…"` 로 중복·불필요 노트를 숨김 처리(행은 보존, audit용).
- `sdi agent-note list --scope plan --anchor <ID>` 로 특정 앵커의 활성 노트 조회.

## 시스템 흐름

에이전트 A가 관찰 게시 → `sdi agent-note append` → 데몬이 앵커·종류별로 저장 → 에이전트 B가 `sdi agent-note list` 로 맥락 수신 → 에이전트 A가 `--kind handoff --to B` 로 제어 이전 → 에이전트 B가 `handoffs B` 로 확인 → 작업 후 `ack` → 대기 큐에서 제거. 통신 흐름은 자율성 모드와 무관하게 항상 열려 있다.

## 어디에 구현되어 있나

`/agent-note` 슬래시 명령(`plugin/commands/agent-note.md`)이 전체 인터페이스를 정의한다. `agent-note` 스킬(`plugin/commands/agent-note.md`)이 용도별 적용 지침을 제공한다. 데몬이 노트 저장·조회·ack·retire를 처리하고, kind/scope/anchor 인덱스로 빠른 조회를 지원한다.

## 미확정 (OPEN)

- [ ] OPEN: `sdi agent-note list` 에서 `retire` 된 노트가 정말로 숨겨지는지(soft delete), audit 모드로 볼 수 있는 API가 있는지 확인 필요.
