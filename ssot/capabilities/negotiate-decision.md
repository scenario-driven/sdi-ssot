---
id: capability.negotiate-decision
kind: Capability
title: 의사결정 로그 기록 및 조회
purpose: "왜 X 대신 Y를 선택했는지를 ADR 방식의 추가 전용 의사결정 로그로 남기고, 기존 결정을 수퍼시드 체인으로 갱신한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/decide.md
relatesTo:
  - { to: concept.decision, type: mutates, note: "새 결정을 추가하거나 기존 결정을 수퍼시드 체인으로 연결한다" }
  - { to: concept.plan, type: reads, note: "결정은 플랜에 귀속되며 플랜의 의사결정 맥락을 구성한다" }
  - { to: concept.scenario, type: relates-to, note: "시나리오나 요구사항 변경 이유는 결정 로그에 기록한다(본문 변경 이력 금지)" }
  - { to: concept.autonomy-policy, type: relates-to, note: "자율성 모드에 따라 결정 적용 전 사람 확인 여부가 달라진다" }
impacts:
  - concept.decision
  - concept.plan
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

"왜 이렇게 했나"를 남기는 것은 SDI에서 의사결정 로그(/decide)만의 역할이다. 플랜·시나리오·요구사항 본문에 "예전엔 A였으나 이제 B"를 쓰는 것은 금지된다(D12 스냅샷 원칙). 그 이유는 결정 로그에 추가하는 방식으로만 보존한다.

결정은 추가 전용이다. 잘못된 결정을 발견하면 수정하는 것이 아니라 "수퍼시드" 새 결정을 만들어 이전 결정을 `superseded` 상태로 전환한다. 이렇게 쌓인 수퍼시드 체인이 결정 이력이다. 컨텍스트·결정·결과라는 세 항목이 결정 본문의 표준이다.

## 행위

- `sdi decision create <PLAN-ID> <SHORT-CODE> "<title>" --body "$(cat body.md)"` 로 새 결정을 등록한다.
- `sdi decision supersede <PRIOR-DEC-ID> --plan-id <ID> --short-code <NEW> --title "<title>" --body "…"` 로 이전 결정을 수퍼시드한다.
- `sdi decision list <PLAN-ID>` 와 `sdi decision view <DEC-ID>` 로 조회한다.
- 자율성 모드(L3/L4/L5)에 따라 결정 적용에 사람 확인 게이트가 개입한다.

## 시스템 흐름

의사결정 필요 상황 발생 → `/decide create` 또는 M3 협상을 통한 consensus 결정 생성 → 결정이 `proposal` 단계 → critique 에이전트가 반박 → 합의(consensus) 또는 이견(dissensus) 에미트 → 자율성 모드에 따라 사람 확인 후 적용 → 기존 결정이 잘못됐으면 `supersede`로 새 결정 추가, 이전 결정은 `superseded` 상태로.

## 어디에 구현되어 있나

`/decide` 슬래시 명령(`plugin/commands/decide.md`)이 생성·수퍼시드 절차를 정의한다. MCP 도구 `add_decision` 이 LLM 주도 등록 경로를 제공한다. MCP 도구 `get_recent_decisions` 로 최신 ADR 로그를 조회할 수 있다. 데몬이 추가 전용 불변성과 수퍼시드 체인을 강제한다.

## 미확정 (OPEN)

- [ ] OPEN: 결정 적용 시 L4 forced 조건(architecture / schema / naming-canonical 종류) 검사가 decide 명령에서 직접 처리되는지, autonomy 데몬 레이어에서만 처리되는지 확인 필요.
