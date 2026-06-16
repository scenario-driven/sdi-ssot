---
id: integration.claude-code-skills
kind: Integration
title: Claude Code 스킬 연동
purpose: LLM이 작업 문맥에 따라 SDI 고유 절차(엔티티 오리엔테이션·GWT 변환·라운드 수명주기·증거 기록)를 자기 컨텍스트에 직접 로드하도록 — 플러그인이 네 개 스킬을 Claude Code 마켓플레이스 규약으로 등록해 `/`로 호출 가능하게 한다.
definition: "SDI 플러그인의 `plugin.json`이 네 개 스킬(sdi-overview·sdi-scenario·sdi-round·sdi-evidence)을 `skillsList`에 선언한다. 각 스킬은 `SKILL.md` 파일로 구성되며, SessionStart 훅의 설치 게이트가 세 파일의 존재를 실행 전에 검증한다(skill-file-integrity)."
integratesWith: []
implementedIn:
  - sdi-plugin/plugin/.claude-plugin/plugin.json
  - sdi-plugin/plugin/skills/sdi-overview/SKILL.md
  - sdi-plugin/plugin/skills/sdi-scenario/SKILL.md
  - sdi-plugin/plugin/skills/sdi-round/SKILL.md
  - sdi-plugin/plugin/skills/sdi-evidence/SKILL.md
  - sdi-plugin/plugin/adapters/shared/sdi-hooks.cjs
impacts:
  - domain.plugin-runtime
  - domain.scenario-management
  - domain.round-execution
relatesTo:
  - to: domain.plugin-runtime
    type: realizes
    note: 스킬 등록이 플러그인 런타임 컨텍스트 공급의 일부다
  - to: domain.scenario-management
    type: relates-to
    note: sdi-scenario 스킬이 시나리오 관리 도메인 진입을 안내한다
  - to: domain.round-execution
    type: relates-to
    note: sdi-round 스킬이 라운드 수명주기 흐름을 담는다
governedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 Claude Code의 스킬 시스템이다. Claude Code는 플러그인의 `plugin.json`에 선언된 스킬을 로드해, 사용자나 LLM이 해당 스킬을 호출할 때 `SKILL.md` 본문을 컨텍스트로 주입한다.

SDI는 네 개 스킬을 등록한다.

- **sdi-overview**: SDI의 다섯 1등 시민 엔티티(Plan·Requirement·Scenario·Decision·Round), 정규 워크플로우, 각 상황별 올바른 도구 선택 기준, MCP 도구 맵, 실패 코드 복구 가이드를 담는다. 처음 SDI 프로젝트에서 작업을 시작할 때 읽는 오리엔테이션 문서다.
- **sdi-scenario**: 자유 형식 자연어 → D5 준수 Given/When/Then 변환 기준을 담는다. 에이전트가 시나리오를 작성하거나 검토할 때 참조한다.
- **sdi-round**: 라운드 생성·활성화·태스크 분해 흐름, strict-regression 기본 동작, 진행 중 태스크 정책(D10), 파괴 분석 진입 조건을 담는다.
- **sdi-evidence**: 태스크 완료 시 구조화된 증거(code·test·ci·external·transcript 종류) 기록 방식을 담는다. 자유 형식 evidence 문자열이 거부되는 이유와 올바른 형식을 설명한다.

## 구현 위치 (provenance)

스킬 등록 선언은 `plugin/.claude-plugin/plugin.json`의 `skillsList` 배열이다. 이 배열은 각 스킬의 이름·파일 경로·설명을 기술한다. 실제 스킬 내용은 `plugin/skills/<name>/SKILL.md` 파일 하나다.

설치 게이트(`plugin/adapters/shared/sdi-hooks.cjs`의 `verifySdiSkills`)는 SessionStart 시점에 네 스킬 파일이 모두 존재하는지 확인한다. 하나라도 없으면 경고를 출력하고 설치 게이트가 실패 처리한다. 이 세 파일(`plugin.json`의 `skillsList` 엔트리, `SKILL.md` 파일, `verifySdiSkills` 체크 목록)은 같은 커밋에서 동시에 갱신되어야 하는 3방향 잠금 계약이다.

## 불변식

- 네 스킬 파일이 모두 존재해야 설치 게이트를 통과한다. SessionStart가 스킬 파일 부재를 감지하면 플러그인 재설치를 안내한다.
- 스킬 파일은 `plugin/` 디렉토리에만 위치한다. 사용자 데이터 XDG 경로에 스킬 파일을 쓰는 일은 없다(LM-8 경로 분리 불변식).
- `plugin.json`의 `skillsList`와 실제 `SKILL.md` 파일, 훅의 검증 목록은 항상 동기화되어야 한다. 하나만 추가/삭제하면 설치 게이트가 실패한다.

## 영향 범위

스킬 파일이 없거나 손상되면 설치 게이트가 SessionStart에서 실패하고 SDI 전체가 동작하지 않는다. 스킬 자체는 컨텍스트 제공 역할이므로 없어도 MCP나 훅 기능은 작동하지만, 설치 게이트의 무결성 검사가 전부를 막는다. 따라서 스킬 파일은 실질적으로 플러그인 배포 무결성의 카나리아 역할을 한다.

## 미확정 (OPEN)
- [ ] OPEN: 스킬 파일의 정확한 버전 관리 방식 — 스킬 내용이 바뀔 때 `plugin.json` 버전 갱신이 필요한지, 별도 스킬 버전 필드가 있는지 확인 필요.
