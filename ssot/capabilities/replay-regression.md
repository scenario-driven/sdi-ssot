---
id: capability.replay-regression
kind: Capability
title: 회귀 자동 재검증(Replay)
purpose: "이전 라운드에서 통과한 시나리오의 증거를 다음 라운드 활성화 시 자동으로 재확인해 회귀 신호를 사람의 별도 지시 없이 유지한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-round/SKILL.md
relatesTo:
  - { to: concept.regression, type: mutates, note: "strict-regression 모드에서 이전 판정이 새 라운드로 자동 이월된다" }
  - { to: concept.round-baseline, type: reads, note: "이전 라운드의 판정 집합이 이번 라운드의 기준선이 된다" }
  - { to: concept.round, type: reads, note: "R2+ 활성화 시 이월 로직이 동작한다; R1에서는 적용 불가" }
  - { to: concept.scenario, type: reads, note: "passed 판정 시나리오의 evidence ref를 재확인해 여전히 passing인지 검사한다" }
  - { to: concept.task-evidence, type: reads, note: "이전 라운드에 기록된 증거 ref가 재검증의 입력이 된다" }
  - { to: concept.convergence, type: relates-to, note: "회귀가 0건 유지되는 것이 라운드 간 수렴의 지표다" }
impacts:
  - concept.regression
  - concept.round
  - concept.scenario
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

R2 이후 라운드를 활성화하면 데몬이 이전 라운드의 시나리오 판정을 자동으로 이번 라운드에 가져온다. 이전에 통과(passing)했던 시나리오는 이번 라운드에서도 통과 상태로 시작하되, 기록된 증거 ref를 재확인해 더 이상 유효하지 않으면 판정이 낮아진다. 실패(failing)했던 시나리오는 이번 라운드에서도 failing으로 남아—새 증거를 제출하지 않는 한—계속 검증 대상이 된다. 사람이 별도로 "회귀 테스트 실행"을 지시할 필요가 없다.

이것이 SDI가 TDD/BDD를 넘어서는 핵심이다. 회귀 검증을 별도 파이프라인이나 테스트 스위트가 아니라 라운드의 기본 동작으로 내장한다.

## 행위

- R2+ 라운드를 `--mode strict-regression`(기본)으로 만들어 이월을 활성화한다.
- `sdi round activate <ROUND-ID>` 시 데몬이 이전 판정을 새 라운드 결과 테이블에 복사한다.
- 이전에 통과한 시나리오의 증거 ref를 자동으로 재확인해 끊긴 ref는 판정을 낮춘다.
- `forward-only` 모드를 명시하면 이월 없이 새 시나리오만 검증한다(의도적 선택, 이유를 결정 로그에 남긴다).

## 시스템 흐름

`sdi round create <PLAN-ID> R2 --mode strict-regression` → `sdi round activate <ROUND-ID>` → 데몬이 R1 판정 이월: passed는 이번 라운드 passed로 복사 + ref 재확인, failed는 이번 라운드 failed로 복사, blocked는 needs-attention으로 표시 → `scenarios_needing_verification`에 새 시나리오 + 이월 failed/blocked 시나리오 포함 → LLM이 필요한 태스크만 분해해 구현·증거 갱신.

## 어디에 구현되어 있나

strict-regression 이월 로직은 `sdi-round` 스킬(`plugin/skills/sdi-round/SKILL.md`)과 데몬의 라운드 활성화 HTTP 핸들러에 정의된다. 웹 대시보드 TimelineView가 라운드 간 판정 이력을 시각화한다.

## 미확정 (OPEN)

- [ ] OPEN: 데몬이 code 종류 증거 ref 재확인 시 실제 파일 시스템에 접근하는지, 아니면 ref 문자열이 비어있지 않은지만 확인하는지 구현 상세 확인 필요.
