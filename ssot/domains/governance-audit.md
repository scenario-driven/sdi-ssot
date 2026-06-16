---
id: domain.governance-audit
kind: Domain
title: 거버넌스 감사 (Governance Audit)
purpose: "에이전트의 모든 게이트 통과·차단·bypass·자율 적용을 감사 로그로 기록하고, 훅 bypass 사용과 프로토콜 위반을 투명하게 만들어 자율 실행의 신뢰 기반을 구축한다."
definition: "활동 감사 이벤트 기록과 bypass 메커니즘(`sdi bypass arm`)을 담당하는 경계. 모든 PreToolUse 게이트(통과·차단), 자율 결정 적용, circuit breaker 작동, bypass 사용이 audit 이벤트로 저장된다. bypass는 단일 도구 호출 한 번만 허용하는 일회성 마커로, 매 사용이 감사된다."
servesPersona:
  - persona.solo-builder
relatesTo:
  - to: domain.autonomy
    type: relates-to
    note: 자율 모드 변경과 결정 자동 적용이 모두 감사 이벤트로 기록된다.
  - to: domain.plugin-runtime
    type: relates-to
    note: PreToolUse 훅이 게이트 통과·차단 감사 이벤트를 생성하는 주체다.
  - to: domain.collaboration-patterns
    type: relates-to
    note: direct 패턴 사용(안티패턴 마커)이 audit=anti-pattern-direct로 기록된다.
  - to: concept.run
    type: relates-to
    note: 에이전트 실행 단위(AgentRun)별 감사 이력을 추적한다.
governedBy: []
realizedBy: []
impacts:
  - domain.autonomy
implementedIn:
  - "sdi-plugin/crates/daemon/src/router/run.rs"
  - "sdi-plugin/crates/core/src/run.rs"
  - "sdi-plugin/crates/daemon/src/events.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 "에이전트를 믿을 수 있는가"의 근거를 만든다. 에이전트가 자율적으로 결정을 적용하고 코드를 변경하는 SDI 환경에서, "무슨 일이 있었는가"를 추적하지 않으면 신뢰가 형성되지 않는다. 모든 게이트 통과·차단·bypass·자율 적용을 감사 이벤트로 기록해 빌더가 언제든 확인할 수 있게 한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **감사 이벤트 종류**:
  - 게이트 통과 / 차단 (D21·활성태스크·D26·D29 각각)
  - 자율 결정 자동 적용 (L5)
  - circuit breaker 작동
  - bypass 사용 (`audit=manual-override`)
  - 프로젝트 비활성화 상태에서 게이트 통과 (`audit=project-disabled`)
  - direct 패턴 자동 부여 (`audit=anti-pattern-direct`)

- **bypass 메커니즘**: `sdi bypass arm --reason "<사유>"`가 XDG cache에 일회성 마커를 쓴다. 마커는 다음 한 번의 변경 도구 호출 후 자동 삭제된다. TTL 기본 60초. 만료된 마커는 정리되고 게이트는 열리지 않는다. 모든 bypass 무장과 소비가 감사 이벤트로 기록된다. 일상적 bypass는 프로토콜 위반이다.

- **SSE 이벤트 스트림**: `sdid`가 `/events` 엔드포인트로 실시간 활동 이벤트를 스트림한다. 대시보드가 이를 구독해 실시간으로 진행 상황과 감사 이벤트를 표시한다.

포함되지 않는 것: 결정 내용 자체(domain.decision-negotiation), 자율 정책 설정(domain.autonomy), 시나리오 상태(domain.scenario-management).

## 기능

- 게이트 통과·차단 감사 이벤트를 기록한다.
- 자율 결정 적용 감사 이벤트를 기록한다.
- bypass 마커를 발행하고 소비 후 삭제한다.
- SSE로 실시간 감사 이벤트를 스트림한다.
- 빌더가 감사 이력을 조회할 수 있는 API를 제공한다.

## 시스템 흐름

에이전트가 도구를 호출하면 PreToolUse 훅이 게이트를 검사하고 결과(통과/차단/bypass)를 데몬에 기록한다. 데몬은 이 이벤트를 SSE로 브로드캐스트하고 대시보드가 실시간 표시한다. 빌더가 타임라인 뷰에서 "어떤 에이전트가 어떤 도구를 호출했고 게이트가 어떻게 처리했는지"를 확인한다.

## 다른 도메인과의 관계

거버넌스 감사는 SDI 전체의 투명성 계층이다. 각 도메인이 "무엇을 할 것인가"를 결정하면, 이 도메인이 "그것이 실제로 어떻게 일어났는가"를 기록한다. 빌더가 에이전트의 자율 실행을 신뢰하는 근거는 이 감사 이력이다.

## 미확정 (OPEN)
- [ ] OPEN: 감사 이벤트의 영구 보존 정책(얼마나 오래 보관하는지, 삭제 정책이 있는지) 코드 또는 PRD에서 확인 필요.
