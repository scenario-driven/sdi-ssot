---
id: capability.track-task-evidence
kind: Capability
title: 태스크 증거 기록
purpose: "태스크를 완료로 전환할 때 체크 가능한 증거를 구조화해 등록함으로써 라운드 판정과 회귀 자동 검증의 신뢰성을 확보한다."
servesPersona:
  - persona.coding-agent
  - persona.subagent
implementedIn:
  - sdi-plugin/plugin/skills/sdi-evidence/SKILL.md
relatesTo:
  - { to: concept.task-evidence, type: mutates, note: "task done 전환 시 증거 항목을 구조화해 기록한다" }
  - { to: concept.task, type: mutates, note: "증거가 있어야 done 전환이 허용된다(EVIDENCE_REQUIRED 강제)" }
  - { to: concept.scenario, type: reads, note: "태스크의 부모 시나리오가 이 증거로 passing 판정을 받는다" }
  - { to: concept.round, type: reads, note: "라운드의 시나리오 판정(verdict)에 증거 ref가 사용된다" }
  - { to: concept.regression, type: relates-to, note: "다음 라운드의 strict-regression 이월 시 기록된 증거 ref를 재확인한다" }
  - { to: concept.evidence-tsv, type: mutates, note: "구조화 증거(kind/ref/summary)가 태스크에 저장되며 TSV 형태로 읽을 수 있다" }
impacts:
  - concept.task-evidence
  - concept.task
  - concept.regression
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

코드 수정이 끝나면 그 작업이 실제로 완료됐음을 입증하는 증거를 남기고 태스크를 완료로 전환한다. 증거 없이 완료를 선언하는 것은 데몬이 거부한다(`EVIDENCE_REQUIRED`). 이 강제가 중요한 이유는 다음 라운드에서 이 증거 ref를 다시 확인해 "여전히 통과하는가"를 자동으로 재검증하기 때문이다. 체크 불가한 증거("신뢰해", "로그 참조")는 자동 회귀 검증을 망가뜨린다.

증거의 종류는 다섯 가지다: 코드 파일과 행 번호(code), 테스트 이름과 통과 트랜스크립트(test), CI 실행 ID와 URL(ci), 외부 URL과 설명(external), 명령어 출력을 직접 붙여 넣은 내용(transcript). 하나의 태스크에 여러 종류를 함께 등록할 수 있다.

## 행위

- `sdi task done <TASK-ID> --evidence-kind <종류> --evidence-ref "<ref>" --evidence-summary "<요약>"` 으로 완료 전환과 증거를 함께 전달한다.
- 여러 증거는 `--evidence-kind … --evidence-ref … --evidence-summary …` 세 플래그를 순서대로 반복해 첨부한다.
- MCP 도구 `update_task_evidence` 로 증거를 먼저 저장하고, 이후 done 전환을 분리해 수행하는 경로도 있다.
- 파일이 리팩토링으로 이동했으면 증거 ref를 새 경로로 갱신해야 자동 회귀가 올바르게 동작한다.

## 시스템 흐름

구현 완료 → `sdi task done` 또는 MCP `update_task_evidence` + done 전환 → 데몬이 증거 배열 비어있지 않음 검증 → 태스크 상태 `done` 변경 → 부모 시나리오에 `passing` 판정 부여 → 다음 라운드 strict-regression 활성화 시 데몬이 기록된 ref를 재확인해 통과 여부 자동 판정.

## 어디에 구현되어 있나

증거 기록 절차는 `sdi-evidence` 스킬(`plugin/skills/sdi-evidence/SKILL.md`)에 정의된다. MCP 도구 `update_task_evidence` 가 JSON 입력으로 증거를 구조화 저장한다. 웹 대시보드의 BoardView가 태스크 상태와 증거 존재 여부를 시각화한다.

## 미확정 (OPEN)

- [ ] OPEN: 자동 회귀 시 데몬이 code 종류 증거 ref(파일:행)를 실제로 파일 시스템에서 확인하는지, 아니면 ref 문자열 존재 여부만 확인하는지 구현 확인 필요.
