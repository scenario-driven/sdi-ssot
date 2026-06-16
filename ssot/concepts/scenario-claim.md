---
id: concept.scenario-claim
kind: Concept
title: Scenario Claim(시나리오 클레임)
definition: "D29 다중 세션 리소스 클레임. 시나리오가 active인 동안 편집할 파일 경로 글로브 목록을 선점 신고한다. PreToolUse 훅이 편집 전 겹치는 클레임을 감지해 차단하여 여러 에이전트 세션이 같은 파일을 동시에 수정하는 충돌을 방지한다."
relatesTo:
  - { to: concept.scenario, type: belongs-to, note: "클레임은 시나리오에 attached되어 claimed_resources_json과 claim_status 필드에 저장된다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "plan_single_session_lock=true이면 플랜의 시나리오들이 동시에 여러 세션에서 claim 활성화되지 못한다." }
  - { to: domain.agent-coordination, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/scenario.rs
  - sdi-plugin/crates/core/src/pattern.rs
  - sdi-plugin/plugin
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Scenario Claim(시나리오 클레임)은 SDI가 다중 에이전트 세션 환경에서 파일 편집 충돌을 방지하는 메커니즘이다(D29). 하나의 시나리오를 실행 중인 에이전트 세션이 "이 파일들은 내가 편집할 것이다"라고 미리 신고(claim)하면, 다른 세션이 같은 파일을 편집하려 할 때 훅이 이를 감지해 차단한다.

클레임 상태(claim_status)는 네 가지다.

- **none**: 클레임 없음(기본값).
- **requested**: 클레임 요청됨(아직 활성화 대기 중).
- **active**: 클레임 활성 중 — 해당 파일 경로들이 점유됨.
- **released**: 클레임 해제됨.

`claimed_resources_json`은 파일 경로 글로브 패턴의 JSON 배열이다. 유효 규칙은 다음과 같다.

- JSON 배열이어야 한다.
- 각 항목은 비어 있지 않아야 한다.
- 순수 와일드카드(예: `**`, `*`, `**/*`)만으로 이루어진 항목은 허용하지 않는다 — 겹침 탐지가 무의미해지기 때문이다.

PreToolUse 훅은 `Edit`/`Write`/`NotebookEdit` 도구 호출 전 daemon의 `/scenarios/active-claims`를 조회해 편집 대상 경로와 겹치는 활성 클레임이 있으면 `exit code 2`와 구조화된 JSON 오류를 반환한다. daemon이 응답하지 않으면 훅은 차단하지 않고 진행한다(가용성 우선 정책).

긴급 우회는 `sdi bypass arm --reason "<이유>"`로 일회성 bypass 마커를 XDG 캐시에 생성해(기본 TTL 60초) D21 위임, 활성 태스크, D29 클레임 겹침 차단을 한 번만 건너뛸 수 있다.

## 엔티티 (DB)

클레임은 독립 테이블이 없다. scenarios 테이블의 `claimed_resources_json`(TEXT)과 `claim_status`(TEXT) 두 컬럼에 저장된다.

## API 표면

daemon의 `/scenarios/active-claims` 엔드포인트가 현재 active인 모든 클레임 목록을 반환한다. 훅이 이를 조회해 편집 전 충돌 검사를 수행한다.

## 불변식

- **글로브 형식(D29)**: claimed_resources_json은 JSON 배열, 각 항목 비어있지 않음, 순수 와일드카드 금지.
- **daemon 다운 시 비차단**: daemon이 응답하지 않으면 훅은 차단하지 않고 진행.

## 구현 위치 (provenance)

`validate_claimed_resources` 함수와 `ClaimStatus` 열거형은 `sdi-plugin/crates/core/src/scenario.rs`와 `sdi-plugin/crates/core/src/pattern.rs`에 있다. PreToolUse 훅 구현은 `sdi-plugin/plugin/` 하위에 있다.

## 미확정 (OPEN)

- [ ] OPEN: SDI_ACTIVE_SCENARIO 환경변수를 통한 시나리오 바인딩이 daemon의 AgentRun↔Scenario 엣지로 언제 이전되는지 확인 필요(CLAUDE.md에서 "until the daemon gains the AgentRun↔Scenario edge"라고 언급됨).
