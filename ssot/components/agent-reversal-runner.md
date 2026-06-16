---
id: component.agent-reversal-runner
kind: SystemComponent
title: reversal-runner 서브에이전트
purpose: D28 롤백 계획(reversal_plan)을 실제로 집행하는 전문 에이전트다. Decision에 rollback_initiated 이벤트가 발생하면, 기록된 reversal_plan의 종류(migration_sql, git_revert, fs_snapshot, compensating_action)에 따라 실행하고, 롤백 완료를 별도 Decision 행(append-only)으로 감사 기록한다.
realizedBy:
  - domain.decision-negotiation
  - domain.governance-audit
implementedIn:
  - sdi-plugin/plugin/agents/reversal-runner.md
dependsOn:
  - component.daemon
  - component.cli
consumesApi: []
providesApi: []
integratesWith: []
impacts:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.decision-negotiation
    type: backed-by
  - to: domain.governance-audit
    type: backed-by
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 책임

reversal-runner는 입장(stance)이 `neutral`인 에이전트로, 롤백의 실행 담당자다. 설계·협상 없이 이미 결정된 롤백 계획을 순서대로 실행하고 감사 행을 남긴다. 롤백은 자율 수준과 무관하게 항상 실행 가능한 동작(`rollback_initiated` 이벤트는 모든 자율 모드에서 처리된다, D20).

두 가지 경로로 활성화된다. 첫째, 데몬 SSE 스트림에서 `rollback_initiated` 이벤트가 흘러나올 때. 둘째, 다른 에이전트가 AgentNote로 `to_agent=reversal-runner`와 `decision_id`를 전달할 때.

실행 단계는 다음과 같다. 원본 Decision 행에서 `reversal_plan`을 읽는다. 롤백 종류에 따라 집행한다 — `migration_sql`이면 스키마 복구 SQL을 실행하고, `git_revert`이면 커밋 되돌리기 명령을 실행하고, `fs_snapshot`이면 스냅샷에서 파일을 복원하고, `compensating_action`이면 역방향 보상 동작을 수행한다. 실행 완료 후, 원본 Decision을 `reversal_of`로 참조하는 새 Decision 행(kind=consensus)을 append-only로 기록한다. 이 로그가 "무엇이 언제 롤백됐는가"의 감사 흔적이 된다.

## 경계와 의존

Bash·Read·Edit·Write·Skill 도구를 갖는다. 롤백 계획의 종류에 따라 파일 수정이나 Bash 명령 실행이 필요하다. `sdi decision` CLI 명령으로 롤백 완료 감사 행을 기록한다.

## 통신 패턴

SSE 이벤트 또는 AgentNote 핸드오프로 활성화된다. 롤백 완료 후 감사 decision id를 포함한 결과를 메인 세션에 반환한다.

## 하위 서브패키지 (책임 단위)

단일 에이전트 역할 정의 파일(`plugin/agents/reversal-runner.md`)로 구성된다. 트리거 조건·4가지 롤백 종류별 실행 절차·감사 행 기록 불변식이 자연어 프롬프트로 정의되어 있다.

## 미확정 (OPEN)

- [ ] OPEN: `fs_snapshot` 롤백의 스냅샷 생성 시점과 저장 위치(XDG 경로인지 기타인지) 확인 필요.
- [ ] OPEN: 여러 단계로 구성된 reversal_plan에서 중간 단계 실패 시 부분 롤백 처리 방법이 명세되어 있지 않다.
