---
id: capability.check-status
kind: Capability
title: SDI 상태 빠른 확인
purpose: "현재 cwd 의 SDI 프로젝트·활성 플랜·라운드·진행 중 태스크·데몬 헬스를 한 번의 명령으로 요약해 작업 컨텍스트를 빠르게 복원한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
implementedIn:
  - sdi-plugin/plugin/commands/sdi-status.md
  - sdi-plugin/plugin/commands/sdi-status.md
relatesTo:
  - { to: concept.plan, type: reads, note: "활성 플랜 ID와 상태를 cwd 기준으로 조회한다" }
  - { to: concept.round, type: reads, note: "활성 라운드 ID와 모드를 조회한다" }
  - { to: concept.task, type: reads, note: "진행 중 태스크 목록과 상태를 조회한다" }
  - { to: concept.project, type: reads, note: "cwd를 프로젝트에 매핑해 프로젝트 키·이름을 확인한다" }
impacts:
  - concept.plan
  - concept.round
  - concept.task
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 사용자가 할 수 있는 일

새 Claude Code 세션을 열었을 때, 또는 잠시 다른 작업을 하다 SDI 프로젝트로 돌아왔을 때, 현재 무슨 상태인지를 한 번에 파악한다. `/sdi-status`를 실행하면 cwd를 SDI 프로젝트에 매핑하고, 그 프로젝트의 활성 플랜·라운드·진행 중 태스크·데몬 헬스를 짧은 리포트로 보여준다.

이 역량은 순수 읽기다. MCP의 `get_plan_context` 와 비슷하지만 차이가 있다. MCP 도구는 rag 스코프 지식만 보여주지만, `/sdi-status`는 데몬 HTTP 엔드포인트를 직접 호출해 태스크 실행 상태, 데몬 liveness, short code 같이 rag 범위 밖의 운영 정보를 포함한다.

## 행위

- `sdi daemon status` 와 `sdi doctor` 로 데몬 헬스를 확인한다.
- `sdi project by-cwd "$(pwd)"` 로 현재 cwd의 프로젝트를 찾는다.
- `sdi plan active <PROJECT-ID>` 로 활성 플랜을 조회한다.
- `sdi round active <PLAN-ID>` 로 활성 라운드를 조회한다.
- `sdi task list <ROUND-ID>` 로 진행 중 태스크를 조회한다.
- 위 결과를 짧은 요약(프로젝트 키+이름, 플랜 제목+상태, 라운드 ID+모드, 태스크 상태, 데몬 헬스)으로 출력한다.

## 시스템 흐름

`/sdi-status` 호출 → 데몬 헬스 확인 → cwd → 프로젝트 매핑 → 플랜 조회 → 라운드 조회 → 태스크 조회 → 리포트 출력. 데몬이 응답하지 않으면 `sdi daemon start` 를 권고한다. cwd가 어떤 프로젝트에도 등록되지 않았으면 `sdi project create` 를 안내한다.

## 어디에 구현되어 있나

`/sdi-status` 슬래시 명령(`plugin/commands/sdi-status.md`)과 `sdi-status` 스킬(`plugin/commands/sdi-status.md`)이 절차를 정의한다. `sdi doctor` CLI가 XDG 경로 문제와 데몬 헬스를 종합 진단한다. 데몬 HTTP API가 각 엔드포인트를 제공한다.

## 미확정 (OPEN)

- [ ] OPEN: `sdi doctor` 가 진단하는 항목 목록(XDG 경로 외에 추가 항목)이 구현됐는지 CLI 코드 확인 필요.
