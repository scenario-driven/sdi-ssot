---
id: integration.clawket-migration
kind: Integration
title: Clawket v3.0 → SDI 마이그레이션 연동
purpose: Clawket v3.0에서 운영 중이던 프로젝트(Plan·Unit·Cycle·Task·decision artifact)를 SDI의 도메인 모델(Plan·Scenario·Round·Task·Decision)로 이전할 수 있도록 한다. v1 PRD 범위 밖(§9 별도 트랙)이며, 마이그레이션 도구는 아직 구현되지 않았다.
definition: "Clawket v3.0 SQLite(`~/.local/share/clawket/clawket.db`)를 직접 읽어 도메인 매핑표에 따라 SDI 엔티티로 변환하고, Task 본문에서 LLM 보조로 GWT 후보를 추출해 검토 큐에 올리는 마이그레이션 도구. 현재 설계만 존재하며 코드 구현은 없다."
integratesWith: []
implementedIn:
  - sdi-plugin/docs/MIGRATION-clawket-to-sdi.md
impacts:
  - domain.scenario-management
  - domain.planning
  - domain.decision-negotiation
  - domain.knowledge-rag
relatesTo:
  - to: domain.scenario-management
    type: relates-to
    note: Clawket Task 본문을 GWT 시나리오 후보로 변환하는 것이 마이그레이션의 핵심 작업이다
  - to: domain.planning
    type: relates-to
    note: Clawket Plan이 SDI Plan으로 직접 매핑된다
  - to: domain.decision-negotiation
    type: relates-to
    note: Clawket의 type=decision artifact가 SDI Decision 엔티티(append-only)로 격상된다
  - to: domain.knowledge-rag
    type: relates-to
    note: Clawket knowledge entry가 SDI knowledge RAG 도메인으로 이식될 수 있다
governedBy: []
owner: TBD
lifecycle: proposed
confidence: inferred
lastVerified: ""
---

## 무엇과 연동하나

상대는 Clawket v3.0이다. SDI는 Clawket의 직계 후속이지만 별개 도구로 새 GitHub 조직(`scenario-driven`)에서 시작한다. Clawket v3.0 사용자가 축적한 작업 이력(Plan·Unit·Cycle·Task·decision artifact·knowledge entry)을 SDI로 이전하려면 데이터 모델 변환이 필요하다.

도메인 매핑은 다음과 같다:

- **Clawket Plan** → **SDI Plan**: 같은 이름, `draft → active → completed` 수명주기 유지. Plan approve 게이트 의미는 재해석됨(SDI는 시나리오 1개 이상 필수, Clawket은 Task 중심).
- **Clawket Unit** → **SDI Scenario.tag**: 엔티티에서 문자열 태그로 격하. SDI는 Unit 엔티티 자체가 없다(D4).
- **Clawket Cycle** → **SDI Round**: 이름과 의미 모두 변경. Cycle은 Kanban WIP 경계였지만 Round는 회귀 검증 파도다.
- **Clawket Task** → **SDI Task(런타임 산출물)**: 수명주기 상태는 유지하되, SDI Task는 LLM이 시나리오를 보고 분해하는 런타임 산출물이다. 사람이 Clawket Task를 직접 SDI Task로 이전하는 것은 아니고, Task 본문에서 GWT 후보를 추출하는 것이 이전 경로다.
- **Clawket type=decision artifact** → **SDI Decision 엔티티**: `scope=rag`에 저장되던 결정 기록이 SDI에서는 1등 시민 엔티티로 격상된다. append-only 이력 + provenance 연결이 추가된다.
- **Clawket knowledge entry(type=doc 등)** → **SDI knowledge(scope=rag)**: 직접 이식 가능한 항목이나 스코프 규칙 재확인 필요.

마이그레이션 도구가 구현되면 수행할 주요 단계는 다음과 같다: Clawket SQLite를 직접 읽기 → 도메인 매핑 적용 → Task 본문에 대해 LLM 보조 GWT 추출(자동 적재 아님, 검토 큐 제출) → decision artifact를 Decision 행으로 변환(`created_at` 보존) → 마이그레이션 로그를 `~/.local/state/sdi/migration.log`에 기록.

## 구현 위치 (provenance)

현재 유일한 근거 문서는 `docs/MIGRATION-clawket-to-sdi.md`다. 마이그레이션 도구의 코드 구현은 v0.1 이후 별도 트랙이다. SDI는 v0.1 수용 기준으로 "빈 SDI DB 위에서 셀프 호스팅"을 정의했으므로, 마이그레이션 없이 신규 SDI 프로젝트를 시작하는 것이 v0.1의 기준 경로다.

## 불변식

- GWT 후보 추출은 자동 적재가 아니라 반드시 사용자 검토 큐를 거쳐야 한다. LLM이 추출한 GWT가 바로 `confirmed=true` 상태로 시나리오 DB에 들어가면 안 된다.
- 마이그레이션 로그는 `~/.local/state/sdi/migration.log`에 단일 감사 채널로 기록되어야 한다.
- Clawket DB(`~/.local/share/clawket/clawket.db`)는 읽기 전용으로 접근해야 한다. 마이그레이션 과정에서 Clawket 데이터를 수정해서는 안 된다.

## 영향 범위

이 연동이 없어도 SDI는 신규 프로젝트에서 완전히 동작한다. Clawket 이력을 SDI로 가져오려는 기존 사용자만 영향을 받는다. 이전이 잘못되면 SDI DB에 GWT 형식이 맞지 않는 시나리오가 들어가거나 결정 이력이 누락될 수 있다.

## 미확정 (OPEN)
- [ ] OPEN: 마이그레이션 도구 구현 시점 미정 — v0.1 이후 별도 트랙(PRD §9).
- [ ] OPEN: Clawket knowledge entry 중 SDI RAG로 이식 가능한 스코프 기준(type·scope 필터) 미정.
- [ ] OPEN: 다중 Clawket 프로젝트를 SDI 프로젝트로 병합·분리하는 시나리오에서 Plan·Task 키 충돌 처리 방안 미정.
