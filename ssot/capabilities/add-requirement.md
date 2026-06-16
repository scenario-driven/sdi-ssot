---
id: capability.add-requirement
kind: Capability
title: 요구사항 등록 및 조회
purpose: "시나리오의 형태가 되지 않는 제약 조건·인터페이스 계약·입력 사실을 요구사항으로 기록해 플랜의 맥락을 보강한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/req.md
relatesTo:
  - { to: concept.requirement, type: mutates, note: "요구사항을 생성하거나 내용을 덮어쓴다(스냅샷 의미론)" }
  - { to: concept.plan, type: reads, note: "요구사항은 특정 플랜에 귀속된다" }
  - { to: concept.scenario, type: relates-to, note: "요구사항은 시나리오를 형성하는 전제 조건이지, 시나리오 자체가 아니다" }
  - { to: concept.task, type: relates-to, note: "라운드 활성화 시 LLM이 요구사항도 참조해 태스크를 분해한다" }
impacts:
  - concept.requirement
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

"데몬은 127.0.0.1 에만 바인딩한다", "Node 20 이상이 필요하다", "외부 결제 API는 idempotency key를 필수로 요구한다"처럼 시스템이 반드시 따라야 하는 제약이나 사실을 기록한다. 이것들은 동작(behavior)이 아니기 때문에 G/W/T 형식으로 쓸 수 없다. 요구사항은 시나리오를 설계할 때 고려해야 할 전제이고, 라운드 활성화 시 LLM이 태스크를 분해할 때 참조한다.

요구사항 본문은 스냅샷(덮어쓰기) 방식이다. 내용이 바뀌면 새 버전을 덧붙이지 않고 현재 내용을 교체한다. "왜 이렇게 결정했는가"라는 변경 이유는 요구사항이 아니라 `/decide`(의사결정 로그)에 기록한다.

## 행위

- `sdi req create <PLAN-ID> <SHORT-CODE> "<title>"` 로 요구사항을 등록한다.
- `--body` 로 마크다운 본문, `--source` 로 근거(파일:행, 티켓 URL 등)를 첨부한다.
- `sdi req list <PLAN-ID>` 와 `sdi req view <REQ-ID>` 로 조회한다.
- 내용이 바뀌면 동일 ID를 덮어써 현재 상태만 유지한다.

## 시스템 흐름

사용자가 제약·사실을 제공 → `/req` 슬래시 명령이 `sdi req create` CLI를 호출 → 데몬이 해당 플랜(상태 무관, 어떤 상태의 플랜이든 수용)에 요구사항 행 저장 → MCP `add_requirement` 도구를 통해 LLM이 직접 등록하는 경로도 동일. 라운드 활성화 시 LLM은 플랜의 요구사항 목록을 읽어 태스크 분해에 반영한다.

## 어디에 구현되어 있나

`/req` 슬래시 명령(`plugin/commands/req.md`)이 생성·조회 절차를 정의한다. MCP 도구 `add_requirement`는 LLM이 직접 구동할 때 동일 등록 경로를 제공한다. 데몬 HTTP API가 실제 저장 및 조회를 처리한다.

## 미확정 (OPEN)

- [ ] OPEN: 요구사항 업데이트(덮어쓰기) 전용 subcommand(`sdi req update`)가 별도로 존재하는지 CLI 구현 확인 필요.
