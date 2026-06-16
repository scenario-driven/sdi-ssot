---
id: decision.clawket-successor-new-org
kind: Decision
title: SDI는 Clawket v4가 아니라 새 조직에서 새 정체성으로 짓는 별도 도구다
purpose: "Clawket의 문제를 해결할 때 기존 Clawket repo를 v4로 업그레이드할지, 새 GitHub 조직에서 새 도구로 시작할지"
definition: "SDI는 Clawket v4가 아니다. Clawket 운영에서 얻은 통찰로 새 GitHub 조직(scenario-driven)에서 새 정체성을 갖는 별도 도구를 만든다. Clawket에서 SDI로의 마이그레이션은 별도 트랙으로 다룬다."
relatesTo:
  - { to: platform.sdi, type: impacts, note: "SDI의 독립 정체성이 이 결정 위에서 성립한다" }
  - { to: domain.project-management, type: impacts, note: "Clawket 사용자의 데이터 마이그레이션 경로가 이 결정에 의해 별도 트랙으로 분리된다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

Clawket v3.0은 지난 약 1개월간 운영되며 LLM 네이티브 작업 관리 도구로 동작했다. 운영 과정에서 네 가지 한계가 명확해졌다. Jira 인간 중심 모델 답습, 자유 형식 evidence로 인한 자동 회귀 불가, 수동 회귀 검증, scope=rag artifact의 구조적 추적 부재. 이 한계들은 Task 중심 설계라는 근본 구조에서 비롯되었다.

이 한계를 해결하는 방법으로 두 가지가 있었다. 하나는 Clawket의 기존 구조 위에서 v4를 올리는 것이고, 다른 하나는 새로운 정체성과 설계 원칙으로 완전히 새로운 도구를 만드는 것이다.

## 결정 (Decision)

Clawket v4가 아니라 새 GitHub 조직 `scenario-driven`에서 `SDI(Scenario-Driven Implementation)`라는 새 정체성으로 별도 도구를 만든다. Clawket이 Task 중심 모델을 답습한 것과 달리, SDI는 GWT 시나리오를 1등 시민으로 두는 근본적으로 다른 설계 원칙에서 출발한다.

Clawket의 기술적 자산(로컬 SQLite + Rust 데몬 + CLI + MCP + 웹 대시보드 구조)은 SDI가 계승하되, 도메인 모델·워크플로우·정체성은 재설계한다. Clawket에서 SDI로의 사용자 마이그레이션은 별도 트랙으로 다루며 본 도구의 v1 범위에 포함하지 않는다.

## 근거와 결과 (Consequences)

새 조직에서 시작하는 이유는 두 가지다.

첫째, 정체성의 단절이 필요하다. Clawket의 Task/Unit/Cycle 개념이 남아있으면 설계 의사결정 시 기존 모델과의 호환성 압박이 생겨 근본적 재설계가 어려워진다. 새 조직·새 repo는 이 압박을 구조적으로 차단한다.

둘째, Clawket은 자신의 도메인에서 계속 유지보수된다. SDI가 Clawket을 대체 선언하지 않으므로, Clawket 사용자는 자신의 페이스에 맞춰 이전을 결정할 수 있다.

대가는 초기 설정 비용이다. 새 GitHub 조직, 새 CI, 새 배포 파이프라인, 새 플러그인 등록이 필요하다. 그러나 이 비용은 일회성이며, 장기적으로 두 도구의 개념적 충돌을 피하는 이점이 크다.

<!-- provenance: sdi-plugin/docs/PRD.md §0 메타 "선행 도구: Clawket v3.0 ... 이름: 미정(가칭 시나리오 엔진) ... 새 GitHub 조직: 미정". scenario-driven/CLAUDE.md "Clawket 의 v4 가 아니라, Clawket 으로부터 얻은 통찰로 별도 새 도구를 새 org 에서 짓는다. 마이그레이션은 별도 트랙(PRD §9)". clawket/plans/scenario-engine-prd.md §0 메타. -->
